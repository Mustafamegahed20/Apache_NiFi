## 10_wait_notify.md

```markdown
```
# Wait/Notify Pattern in Apache NiFi

## Overview

The Wait/Notify pattern in Apache NiFi enables coordination between different flows by allowing one flow to wait for a signal from another flow before continuing. This is essential for orchestrating complex workflows, implementing dependencies, and ensuring proper execution order.

## Core Concepts

### What is Wait/Notify?

**Wait Processor:** Pauses FlowFiles until a specific signal is received
**Notify Processor:** Sends signals to release waiting FlowFiles

### Use Cases

✅ **Coordinate dependent workflows**
- Wait for file processing to complete before loading
- Ensure data quality checks pass before downstream processing
- Synchronize parallel processing branches

✅ **Implement approval workflows**
- Wait for manual approval before proceeding
- Hold processing until external validation completes

✅ **Batch coordination**
- Collect all files before processing
- Wait for all partitions to complete

✅ **External system dependencies**
- Wait for external API to be ready
- Synchronize with scheduled jobs

## Wait Processor

### Configuration

```properties
Release Signal Identifier: ${signal.id}
Target Signal Count: 1
Signal Counter Name: counter
Wait Buffer Count: 1
Releasable FlowFile Count: 1
Expiration Duration: 10 min
Distributed Cache Service: DistributedMapCacheClientService
```

### Key Properties

**Release Signal Identifier:**
- Unique identifier for the signal
- Can use expressions: `${file.name}`, `${batch.id}`
- Must match Notify's identifier

**Target Signal Count:**
- Number of signals required to release
- Use `1` for single signal
- Use `> 1` for multiple signals (e.g., waiting for 3 files)

**Expiration Duration:**
- Maximum wait time
- FlowFiles route to 'expired' after timeout
- Format: `10 min`, `1 hour`, `30 sec`

## Notify Processor

### Configuration

```properties
Release Signal Identifier: ${signal.id}
Signal Counter Name: counter
Signal Buffer Count: 1
Distributed Cache Service: DistributedMapCacheClientService
```

### How It Works

1. Notify writes signal to distributed cache
2. Wait checks cache for matching signal
3. When signal found, Wait releases FlowFile
4. Signal is consumed/removed from cache

## Distributed Cache Service

### Required Setup

**DistributedMapCacheServer:**
```properties
Port: 4557
SSL Context Service: (optional)
```

**DistributedMapCacheClientService:**
```properties
Server Hostname: localhost
Server Port: 4557
Communications Timeout: 30 sec
```

### Clustered Setup

For NiFi cluster:

**On one node (primary):**
```properties
DistributedMapCacheServer:
  Port: 4557
```

**On all nodes:**
```properties
DistributedMapCacheClientService:
  Server Hostname: nifi-primary.example.com
  Server Port: 4557
```

## Basic Patterns

### Pattern 1: Simple Wait/Notify

**Scenario:** Wait for file processing to complete

```
Flow 1 (Processing):
GetFile
  ↓
UpdateAttribute
  signal.id: process-complete-${filename}
  ↓
PutFile
  ↓
Notify
  Release Signal Identifier: ${signal.id}

Flow 2 (Waiting):
GenerateFlowFile
  ↓
UpdateAttribute
  signal.id: process-complete-${expected.filename}
  ↓
Wait
  Release Signal Identifier: ${signal.id}
  Target Signal Count: 1
  Expiration Duration: 5 min
  ↓ success
[Continue processing]
  
  ↓ expired
LogAttribute (timeout - file not processed)
```

### Pattern 2: Multiple Signals

**Scenario:** Wait for 3 partitions to complete

```
Flow 1 (Process Partitions):
GetFile
  ↓
SplitText (3 partitions)
  ↓
UpdateAttribute
  batch.id: batch-${UUID()}
  partition.id: ${fragment.index}
  ↓
[Process partition]
  ↓
Notify
  Release Signal Identifier: ${batch.id}
  Signal Counter Name: partition-counter

Flow 2 (Wait for All):
GenerateFlowFile (trigger)
  ↓
UpdateAttribute
  batch.id: [from somewhere]
  ↓
Wait
  Release Signal Identifier: ${batch.id}
  Target Signal Count: 3
  Signal Counter Name: partition-counter
  ↓ success (all 3 partitions done)
[Aggregate results]
```

### Pattern 3: Approval Workflow

**Scenario:** Wait for manual approval before proceeding

```
Flow 1 (Request Approval):
GetFile (request)
  ↓
UpdateAttribute
  approval.id: approval-${UUID()}
  ↓
InvokeHTTP (send approval request)
  URL: http://approval-system/api/request
  ↓
Wait
  Release Signal Identifier: ${approval.id}
  Expiration Duration: 24 hours
  ↓ success (approved)
[Continue processing]
  
  ↓ expired (no approval in 24h)
PutFile (expired requests)

Flow 2 (Approval Callback):
ListenHTTP (approval webhook)
  ↓
EvaluateJsonPath
  approval.id: $.approvalId
  approval.status: $.status
  ↓
RouteOnAttribute
  ${approval.status:equals('approved')}
  ↓ true
Notify
  Release Signal Identifier: ${approval.id}
```

## Advanced Patterns

### Pattern 1: Batch File Collection

**Scenario:** Wait for all files in a batch before processing

```
Flow 1 (File Arrival):
GetFile
  ↓
UpdateAttribute
  batch.date: ${now():format('yyyy-MM-dd')}
  batch.id: batch-${batch.date}
  file.count: 1
  expected.count: 10
  ↓
PutDistributedMapCache (track files)
  Cache Entry Identifier: ${batch.id}-${filename}
  ↓
ExecuteScript (increment counter)
```

**Groovy Script (Increment Counter):**
```groovy
import org.apache.nifi.distributed.cache.client.DistributedMapCacheClient

def flowFile = session.get()
if (!flowFile) return

def cacheService = context.getProperty("Cache Service")
    .asControllerService(DistributedMapCacheClient)

def batchId = flowFile.getAttribute("batch.id")
def counterKey = "count-${batchId}"

// Get current count
def currentCount = cacheService.get(counterKey, stringSerializer, stringDeserializer)
def count = currentCount ? currentCount.toInteger() + 1 : 1

// Update count
cacheService.put(counterKey, count.toString(), stringSerializer, stringSerializer)

flowFile = session.putAttribute(flowFile, "current.count", count.toString())

// Check if all files received
def expectedCount = flowFile.getAttribute("expected.count").toInteger()
if (count >= expectedCount) {
    // Notify that batch is complete
    def notifyFlow = session.create()
    notifyFlow = session.putAttribute(notifyFlow, "batch.id", batchId)
    session.transfer(notifyFlow, REL_SUCCESS)
}

session.transfer(flowFile, REL_SUCCESS)
```

```
  ↓
Notify (when count reaches expected)
  Release Signal Identifier: ${batch.id}

Flow 2 (Wait for Complete Batch):
GenerateFlowFile (scheduled - check every 5 min)
  ↓
UpdateAttribute
  batch.date: ${now():format('yyyy-MM-dd')}
  batch.id: batch-${batch.date}
  ↓
Wait
  Release Signal Identifier: ${batch.id}
  Expiration Duration: 2 hours
  ↓ success
FetchDistributedMapCache (get all files)
  ↓
MergeContent
  ↓
[Process complete batch]
```

### Pattern 2: External Dependency Check

**Scenario:** Wait for external API to be available

```
Flow 1 (Health Check):
GenerateFlowFile (every 30 sec)
  ↓
InvokeHTTP
  URL: http://external-api/health
  ↓ success (2xx response)
Notify
  Release Signal Identifier: external-api-ready
  ↓ failure
[Retry after 30 sec]

Flow 2 (Main Processing):
GetFile
  ↓
Wait
  Release Signal Identifier: external-api-ready
  Expiration Duration: 5 min
  ↓ success
InvokeHTTP (call external API)
  ↓
[Process response]
  
  ↓ expired
PutFile (api-unavailable/)
```

### Pattern 3: Database Transaction Coordination

**Scenario:** Coordinate multiple database operations

```
Flow 1 (Transaction Initiator):
GetFile
  ↓
UpdateAttribute
  transaction.id: txn-${UUID()}
  ↓
ExecuteSQL (BEGIN TRANSACTION)
  ↓
PutSQL (operation 1)
  ↓ success
UpdateAttribute
  step1.complete: true
  ↓
Notify
  Release Signal Identifier: ${transaction.id}-step1

Flow 2 (Dependent Operation):
[Triggered by some event]
  ↓
UpdateAttribute
  transaction.id: [from somewhere]
  ↓
Wait
  Release Signal Identifier: ${transaction.id}-step1
  Expiration Duration: 30 sec
  ↓ success
PutSQL (operation 2)
  ↓ success
Notify
  Release Signal Identifier: ${transaction.id}-step2

Flow 3 (Transaction Finalizer):
[Trigger]
  ↓
Wait
  Release Signal Identifier: ${transaction.id}-step2
  ↓ success
ExecuteSQL (COMMIT)
  
  ↓ expired (from any Wait)
ExecuteSQL (ROLLBACK)
  ↓
LogAttribute (transaction failed)
```

### Pattern 4: Parallel Processing Synchronization

**Scenario:** Process file in parallel, wait for all to complete

```
Flow 1 (Split and Process):
GetFile (large file)
  ↓
UpdateAttribute
  job.id: job-${UUID()}
  ↓
SplitText (10 partitions)
  fragment.count: 10
  ↓
UpdateAttribute
  partition.id: ${fragment.index}
  ↓
[Heavy Processing - Concurrent Tasks: 10]
  ↓
Notify
  Release Signal Identifier: ${job.id}
  Signal Counter Name: partition-counter

Flow 2 (Aggregator):
[Wait for job trigger]
  ↓
UpdateAttribute
  job.id: [from trigger]
  partition.count: 10
  ↓
Wait
  Release Signal Identifier: ${job.id}
  Target Signal Count: ${partition.count}
  Signal Counter Name: partition-counter
  Expiration Duration: 1 hour
  ↓ success (all partitions done)
FetchDistributedMapCache (retrieve all results)
  ↓
MergeContent
  ↓
[Final aggregation]
  
  ↓ expired
LogAttribute (some partitions failed)
  ↓
InvokeHTTP (alert)
```

### Pattern 5: Multi-Stage Pipeline

**Scenario:** 3-stage pipeline with dependencies

```
Stage 1 (Extract):
GetFile
  ↓
UpdateAttribute
  pipeline.id: pipeline-${UUID()}
  ↓
[Extract Data]
  ↓
PutFile (staging/)
  ↓
Notify
  Release Signal Identifier: ${pipeline.id}-extract-complete

Stage 2 (Transform):
[Trigger]
  ↓
Wait
  Release Signal Identifier: ${pipeline.id}-extract-complete
  Expiration Duration: 10 min
  ↓ success
GetFile (staging/)
  ↓
[Transform Data]
  ↓
PutFile (transformed/)
  ↓
Notify
  Release Signal Identifier: ${pipeline.id}-transform-complete

Stage 3 (Load):
[Trigger]
  ↓
Wait
  Release Signal Identifier: ${pipeline.id}-transform-complete
  Expiration Duration: 10 min
  ↓ success
GetFile (transformed/)
  ↓
PutDatabaseRecord
  ↓
Notify
  Release Signal Identifier: ${pipeline.id}-load-complete

Stage 4 (Cleanup):
[Trigger]
  ↓
Wait
  Release Signal Identifier: ${pipeline.id}-load-complete
  ↓ success
DeleteFile (staging/)
  ↓
DeleteFile (transformed/)
  ↓
UpdateAttribute
  pipeline.status: complete
```

## Signal Patterns

### Pattern 1: Hierarchical Signals

**Use dot notation for organization:**

```properties
Release Signal Identifier: project.module.task.subtask
  Examples:
    - etl.extract.customer.complete
    - etl.transform.customer.complete
    - etl.load.customer.complete
```

### Pattern 2: Timestamped Signals

```properties
Release Signal Identifier: ${task.name}-${now():format('yyyy-MM-dd-HH')}
  Examples:
    - daily-report-2024-01-15-08
    - hourly-sync-2024-01-15-14
```

### Pattern 3: Composite Signals

```properties
Release Signal Identifier: ${environment}-${region}-${service}-${version}
  Examples:
    - prod-us-east-api-v2
    - staging-eu-west-worker-v1
```

## State Management

### Checking Signal State

**ExecuteScript to query cache:**

```groovy
import org.apache.nifi.distributed.cache.client.DistributedMapCacheClient

def flowFile = session.get()
if (!flowFile) return

def cacheService = context.getProperty("Cache Service")
    .asControllerService(DistributedMapCacheClient)

def signalId = flowFile.getAttribute("signal.id")
def counterName = "counter"
def cacheKey = "${signalId}.${counterName}"

// Check if signal exists
def signalValue = cacheService.get(cacheKey, stringSerializer, stringDeserializer)

if (signalValue) {
    flowFile = session.putAttribute(flowFile, "signal.exists", "true")
    flowFile = session.putAttribute(flowFile, "signal.count", signalValue)
} else {
    flowFile = session.putAttribute(flowFile, "signal.exists", "false")
}

session.transfer(flowFile, REL_SUCCESS)
```

### Manual Signal Cleanup

```
ExecuteScript (cleanup old signals)
```

```python
import json
from org.apache.nifi.distributed.cache.client import DistributedMapCacheClient

cache_service = context.getProperty("Cache Service").asControllerService()

# Get all keys (implementation varies by cache type)
# Remove signals older than X hours

flow_file = session.create()
session.transfer(flow_file, REL_SUCCESS)
```

## Debugging Wait/Notify

### Enable Debug Logging

**In logback.xml:**
```xml
<logger name="org.apache.nifi.processors.standard.Wait" level="DEBUG"/>
<logger name="org.apache.nifi.processors.standard.Notify" level="DEBUG"/>
```

### Debugging Flow

```
Flow (Debug):
GenerateFlowFile
  ↓
UpdateAttribute
  signal.id: debug-signal
  ↓
Notify
  Release Signal Identifier: ${signal.id}
  ↓
Wait (10 sec later)
  Release Signal Identifier: ${signal.id}
  ↓ success
LogAttribute (signal received)
  
  ↓ expired
LogAttribute (signal not found - problem!)
```

### Check Cache Contents

```
ExecuteScript (Python):
```

```python
from org.apache.nifi.distributed.cache.client import DistributedMapCacheClient

cache_service = context.getProperty("Cache Service").asControllerService()

# Try to get signal
signal_id = "your-signal-id"
counter_name = "counter"
cache_key = "{}.{}".format(signal_id, counter_name)

serializer = session.stringSerializer
deserializer = session.stringDeserializer

value = cache_service.get(cache_key, serializer, deserializer)

flow_file = session.create()
if value:
    flow_file = session.putAttribute(flow_file, "cache.value", str(value))
    flow_file = session.putAttribute(flow_file, "cache.found", "true")
else:
    flow_file = session.putAttribute(flow_file, "cache.found", "false")

session.transfer(flow_file, REL_SUCCESS)
```

## Performance Considerations

### 1. Cache Performance

**Use Redis for better performance:**

```properties
RedisDistributedMapCacheClientService:
  Redis Mode: Standalone
  Connection String: localhost:6379
  Database Index: 0
  Communication Timeout: 10 sec
  Pool - Max Total: 8
```

### 2. Signal Expiration

**Set TTL on signals:**

```
Notify
  ↓
ExecuteScript (set TTL)
```

```python
# For Redis cache
import redis

r = redis.Redis(host='localhost', port=6379, db=0)
signal_id = flow_file.getAttribute('signal.id')
r.expire(signal_id, 3600)  # Expire in 1 hour
```

### 3. Reduce Wait Polling

```properties
Wait:
  Wait Penalize Duration: 5 sec
  # Reduces CPU usage by not checking too frequently
```

### 4. Batch Notifications

Instead of notifying for each item:

```
[Process Items]
  ↓
MergeContent (batch 100)
  ↓
Notify (once per batch)
  Signal Buffer Count: 100
```

## Error Handling

### Timeout Handling

```
Wait
  Expiration Duration: 10 min
  ↓ expired
LogAttribute
  Log Level: WARN
  ↓
UpdateAttribute
  retry.count: ${retry.count:plus(1)}
  ↓
RouteOnAttribute
  ${retry.count:toNumber():lt(3)}
  ↓ true (retry)
Wait (try again)
  
  ↓ false (max retries)
PutFile (dead letter queue)
  ↓
InvokeHTTP (alert)
```

### Cache Failures

```
Wait/Notify
  ↓ failure (cache unavailable)
LogAttribute (cache error)
  ↓
RouteOnAttribute
  cache.available: ${cache.status:equals('UP')}
  ↓ false
UpdateAttribute
  fallback.mode: true
  ↓
[Alternative processing path]
```

### Orphaned Signals

**Cleanup job:**

```
GenerateFlowFile (daily)
  ↓
ExecuteScript (find old signals)
  # Query cache for signals older than 24h
  ↓
[Delete old signals]
  ↓
LogAttribute (cleanup report)
```

## Real-World Examples

### Example 1: ETL Pipeline Coordination

**Complete ETL with stage dependencies:**

```
Stage 1: Extract (3 sources)
GetFile (Source A)
  ↓
UpdateAttribute
  etl.batch: batch-${now():format('yyyyMMdd')}
  etl.source: A
  ↓
[Extract data]
  ↓
PutFile (staging/source-a/)
  ↓
Notify
  Release Signal Identifier: ${etl.batch}-extract-a

GetFile (Source B)
  ↓
[Similar to Source A]
  ↓
Notify
  Release Signal Identifier: ${etl.batch}-extract-b

GetFile (Source C)
  ↓
[Similar to Source A]
  ↓
Notify
  Release Signal Identifier: ${etl.batch}-extract-c

Stage 2: Wait for All Extracts
GenerateFlowFile (trigger)
  ↓
UpdateAttribute
  etl.batch: batch-${now():format('yyyyMMdd')}
  ↓
Wait (Source A)
  Release Signal Identifier: ${etl.batch}-extract-a
  ↓
Wait (Source B)
  Release Signal Identifier: ${etl.batch}-extract-b
  ↓
Wait (Source C)
  Release Signal Identifier: ${etl.batch}-extract-c
  ↓ all complete
[Continue to Transform]

Stage 3: Transform
GetFile (staging/*)
  ↓
MergeContent
  ↓
[Transform logic]
  ↓
PutFile (transformed/)
  ↓
Notify
  Release Signal Identifier: ${etl.batch}-transform-complete

Stage 4: Load
[Trigger]
  ↓
Wait
  Release Signal Identifier: ${etl.batch}-transform-complete
  ↓
GetFile (transformed/)
  ↓
PutDatabaseRecord
  ↓
Notify
  Release Signal Identifier: ${etl.batch}-load-complete

Stage 5: Final Verification
[Trigger]
  ↓
Wait
  Release Signal Identifier: ${etl.batch}-load-complete
  ↓
ExecuteSQL (validate row counts)
  ↓
RouteOnAttribute (check if valid)
  ↓ valid
Notify
  Release Signal Identifier: ${etl.batch}-complete
  ↓
InvokeHTTP (success notification)
  
  ↓ invalid
[Rollback and alert]
```

### Example 2: File Dependency Management

**Process file only when dependencies are ready:**

```
Main File Arrival:
GetFile (main-file.csv)
  ↓
UpdateAttribute
  file.id: ${filename:substringBeforeLast('.')}
  dependency.1: ${file.id}-metadata.json
  dependency.2: ${file.id}-schema.xsd
  ↓
Wait (for metadata)
  Release Signal Identifier: ${dependency.1}
  Expiration Duration: 1 hour
  ↓ success
Wait (for schema)
  Release Signal Identifier: ${dependency.2}
  Expiration Duration: 1 hour
  ↓ success
[Process with all dependencies]

Dependency 1 Arrival:
GetFile (pattern: *-metadata.json)
  ↓
UpdateAttribute
  dep.id: ${filename}
  ↓
PutFile (metadata/)
  ↓
Notify
  Release Signal Identifier: ${dep.id}

Dependency 2 Arrival:
GetFile (pattern: *-schema.xsd)
  ↓
UpdateAttribute
  dep.id: ${filename}
  ↓
PutFile (schemas/)
  ↓
Notify
  Release Signal Identifier: ${dep.id}
```

### Example 3: Distributed Job Coordination

**Coordinate jobs across multiple NiFi instances:**

```
Instance 1 (Job Initiator):
GenerateFlowFile
  ↓
UpdateAttribute
  job.id: distributed-job-${UUID()}
  total.instances: 3
  ↓
InvokeHTTP (notify instance 2)
  URL: http://nifi-2:8080/trigger
  ↓
InvokeHTTP (notify instance 3)
  URL: http://nifi-3:8080/trigger
  ↓
[Process local partition]
  ↓
Notify
  Release Signal Identifier: ${job.id}
  Signal Counter Name: instance-counter
  ↓
Wait (for all instances)
  Release Signal Identifier: ${job.id}
  Target Signal Count: 3
  Signal Counter Name: instance-counter
  ↓ success
[Aggregate results from all instances]

Instance 2 & 3 (Job Workers):
ListenHTTP (receive trigger)
  ↓
EvaluateJsonPath
  job.id: $.jobId
  ↓
[Process assigned partition]
  ↓
Notify
  Release Signal Identifier: ${job.id}
  Signal Counter Name: instance-counter
```

## Best Practices

### 1. Use Descriptive Signal IDs

✅ **Good:**
```
etl-batch-20240115-extract-customers-complete
report-generation-Q4-2023-approved
data-quality-check-order-pipeline-passed
```

❌ **Bad:**
```
signal1
done
complete
```

### 2. Set Appropriate Timeouts

```properties
# Short-running tasks
Wait:
  Expiration Duration: 5 min

# Long-running batch jobs
Wait:
  Expiration Duration: 2 hours

# Manual approvals
Wait:
  Expiration Duration: 24 hours
```

### 3. Handle Expirations Gracefully

```
Wait
  ↓ expired
LogAttribute
  Log Level: WARN
  ↓
UpdateAttribute
  expired.reason: Signal not received within timeout
  expired.signal: ${signal.id}
  ↓
RouteOnAttribute (check if critical)
  ${critical:equals('true')}
  ↓ true
InvokeHTTP (page on-call)
  
  ↓ false
PutFile (retry queue)
```

### 4. Document Dependencies

Add comments to processors:

```
Wait Processor Comment:
"Waiting for extract-customers signal from upstream ETL.
Timeout: 10 min
Dependency: customers-extract-flow
Expected signal: etl-${batch.date}-extract-customers"
```

### 5. Monitor Signal Health

```
GenerateFlowFile (every 5 min)
  ↓
ExecuteScript (check pending signals)
  # Query cache for signals
  # Count waiting FlowFiles
  ↓
RouteOnAttribute
  ${pending.count:toNumber():gt(100)}
  ↓ true
InvokeHTTP (alert - many pending)
```

### 6. Cleanup Old Signals

```
GenerateFlowFile (daily at 2 AM)
  ↓
ExecuteScript (remove signals > 24h old)
  ↓
LogAttribute (cleanup summary)
```

## Alternatives to Wait/Notify

### When NOT to Use Wait/Notify

❌ **Sequential processing**
- Use RouteOnAttribute instead

❌ **Simple conditional logic**
- Use ExecuteScript or RouteOnAttribute

❌ **File availability checks**
- Use ListFile processor

### Alternative Patterns

**1. Process Groups:**
```
[Input Port]
  ↓
[Process Group 1]
  ↓
[Process Group 2]
  ↓
[Output Port]
```

**2. Funnel Merging:**
```
Flow A ↓
Flow B ↓  → Funnel → [Continue]
Flow C ↓
```

**3. Priority Queues:**
```
Connection Priority Settings:
  High Priority: priorityvalue = '1'
  Low Priority: priorityvalue > '1'
```

## Common Issues and Solutions

### Issue 1: Signal Never Received

**Symptoms:**
- FlowFiles always expire
- No signals in cache

**Solutions:**
```
1. Verify signal IDs match exactly
   Wait:    ${batch.id}
   Notify:  ${batch.id}
   
2. Check cache connectivity
   - Test DistributedMapCache service
   - Verify network connectivity
   
3. Ensure Notify actually runs
   - Check Notify processor stats
   - Verify upstream flow completes
```

### Issue 2: Premature Signal Release

**Symptoms:**
- Wait releases before all work done
- Incomplete processing

**Solutions:**
```
Wait:
  Target Signal Count: 5  # Not 1!
  
Notify:
  # Ensure all 5 tasks call Notify
```

### Issue 3: Cache Saturation

**Symptoms:**
- Slow Wait/Notify performance
- Memory issues

**Solutions:**
```
1. Implement TTL on signals
2. Regular cleanup jobs
3. Use Redis instead of simple cache
4. Monitor cache size
```

### Issue 4: Incorrect Counter Names

**Symptoms:**
- Signals not matching
- Multiple counters created

**Solutions:**
```
# Use consistent counter names
Wait:
  Signal Counter Name: partition-counter
  
Notify:
  Signal Counter Name: partition-counter
  # MUST MATCH EXACTLY
```

## Summary

Wait/Notify pattern enables:
- ✅ Workflow orchestration
- ✅ Dependency management
- ✅ Batch coordination
- ✅ Parallel processing synchronization
- ✅ External system integration

Key Takeaways:
1. Use descriptive signal identifiers
2. Set appropriate expiration timeouts
3. Handle expired FlowFiles gracefully
4. Monitor cache health and performance
5. Implement cleanup for old signals
6. Document dependencies clearly

```
