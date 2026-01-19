
## 12_tailfile_logs.md

```markdown
```
# TailFile and Log Processing in Apache NiFi

## Overview

Log processing is a common use case in NiFi. The TailFile processor enables real-time monitoring of log files, similar to the Unix `tail -f` command. This guide covers collecting, parsing, filtering, and analyzing logs with NiFi.

## Log Processing Processors

### Core Processors

| Processor | Purpose |
|-----------|---------|
| **TailFile** | Monitor and read new lines from files |
| **GetFile** | Read entire log files |
| **ListFile / FetchFile** | List and fetch log files |
| **SplitText** | Split log files into individual lines |
| **ExtractGrok** | Parse logs using Grok patterns |
| **ExtractText** | Extract data using regex |
| **RouteText** | Route based on text content |
| **SplitRecord** | Split structured log records |

## TailFile Processor

### Overview

TailFile monitors files for new content and streams updates in real-time.

### Basic Configuration

```properties
File(s) to Tail: /var/log/application.log
Rolling Filename Pattern: application-${yyyy-MM-dd}.log
State File: ./conf/state/tail-file
Reread on NUL: false
Start Position: Beginning of File
Recursive Lookup: false
Lookup Frequency: 10 sec
Maximum Age: 24 hours
```

### Key Properties Explained

**File(s) to Tail:**
```properties
# Single file
/var/log/app.log

# Multiple files (pipe-separated)
/var/log/app.log|/var/log/error.log

# With wildcards
/var/log/app*.log
```

**Rolling Filename Pattern:**
```properties
# Daily rotation
application-${yyyy-MM-dd}.log

# Hourly rotation
application-${yyyy-MM-dd-HH}.log

# With sequence
application.log.${yyyy-MM-dd}.${seq}
```

**State File:**
- Tracks file position
- Ensures no data loss on restart
- Unique per TailFile processor

**Start Position:**
- `Beginning of File`: Read entire file first time
- `Beginning of Time`: Read all matching files from start
- `Current Time`: Only new content from now

### Basic Examples

**Example 1: Single Application Log**

```
TailFile
  File(s) to Tail: /var/log/myapp/application.log
  Rolling Filename Pattern: application-${yyyy-MM-dd}.log
  ↓
SplitText (one line per FlowFile)
  Line Split Count: 1
  ↓
ExtractText
  Pattern: ^(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}).*?\[(.*?)\].*
  timestamp: $1
  log.level: $2
  ↓
RouteOnAttribute
  ${log.level:equals('ERROR')}
  ${log.level:equals('WARN')}
  ↓ ERROR
PutElasticsearch
  
  ↓ WARN
PutKafka
```

**Example 2: Multiple Log Files**

```
TailFile
  File(s) to Tail: /var/log/app/app.log|/var/log/app/access.log|/var/log/app/error.log
  ↓
UpdateAttribute
  log.source: ${filename}
  ↓
RouteOnAttribute
  ${filename:contains('access')}
  ${filename:contains('error')}
  ${filename:contains('app')}
  ↓ (route to different flows)
```

**Example 3: Directory Monitoring**

```
TailFile
  File(s) to Tail: /var/log/services/*.log
  Recursive Lookup: true
  Lookup Frequency: 10 sec
  Maximum Age: 24 hours
  ↓
UpdateAttribute
  service.name: ${filename:substringBeforeLast('.log')}
  log.path: ${absolute.path}
```

## Log Parsing Patterns

### Apache/Nginx Access Logs

**Log Format:**
```
192.168.1.1 - - [15/Jan/2024:10:30:45 +0000] "GET /api/users HTTP/1.1" 200 1234 "https://example.com" "Mozilla/5.0"
```

**Grok Pattern:**
```
%{IPORHOST:client_ip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:method} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version})?|%{DATA:rawrequest})" %{NUMBER:status_code} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:user_agent}
```

**ExtractGrok Configuration:**
```properties
Grok Expression: ${grok.pattern.apache}
Grok Pattern File: /path/to/patterns.grok
Destination: flowfile-attribute
```

**Flow:**
```
TailFile (/var/log/nginx/access.log)
  ↓
ExtractGrok
  client_ip: extracted
  method: extracted
  request: extracted
  status_code: extracted
  bytes: extracted
  ↓
RouteOnAttribute
  ${status_code:startsWith('5')}  # 5xx errors
  ${status_code:startsWith('4')}  # 4xx errors
  ↓ 5xx
UpdateAttribute
  alert.severity: HIGH
  alert.message: Server error on ${request}
  ↓
InvokeHTTP (alert webhook)
```

### Application Logs (JSON)

**Log Format:**
```json
{"timestamp":"2024-01-15T10:30:45Z","level":"ERROR","logger":"com.example.Service","message":"Connection failed","thread":"pool-1-thread-1","exception":"java.net.ConnectException: Connection refused"}
```

**Flow:**
```
TailFile
  ↓
SplitText (one JSON per line)
  ↓
EvaluateJsonPath
  timestamp: $.timestamp
  log.level: $.level
  logger: $.logger
  message: $.message
  exception: $.exception
  ↓
RouteOnAttribute
  ${log.level:equals('ERROR')}
  ${exception:isEmpty():not()}
  ↓ ERROR with exception
UpdateAttribute
  alert.type: APPLICATION_ERROR
  ↓
PutElasticsearch
  Index: app-errors-${now():format('yyyy.MM.dd')}
```

### Syslog Format

**Log Format:**
```
Jan 15 10:30:45 server1 sshd[12345]: Failed password for root from 192.168.1.100 port 22 ssh2
```

**Grok Pattern:**
```
%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:hostname} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}
```

**Flow:**
```
TailFile (/var/log/syslog)
  ↓
ExtractGrok
  timestamp: extracted
  hostname: extracted
  program: extracted
  pid: extracted
  message: extracted
  ↓
RouteText
  Match Requirement: content must contain match
  Routing Strategy: Route to 'matched' if line matches
  
  ssh_failed_login: (?i)failed password
  sudo_commands: (?i)sudo.*command
  kernel_errors: (?i)kernel.*error
  ↓ ssh_failed_login
UpdateAttribute
  security.event: SSH_FAILED_LOGIN
  severity: MEDIUM
  ↓
PutDatabaseRecord (security_events table)
```

### Custom Application Logs

**Log Format:**
```
2024-01-15 10:30:45.123 [pool-1-thread-1] ERROR com.example.OrderService - Failed to process order 12345: Database connection timeout
```

**ExtractText Configuration:**
```properties
Character Set: UTF-8
Maximum Buffer Size: 1 MB
Enable Repeating Capture Group: false

# Regex pattern
log.pattern: ^(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}) \[(.*?)\] (ERROR|WARN|INFO|DEBUG) (.*?) - (.*)$

# Capture groups
timestamp: $1
thread: $2
level: $3
logger: $4
message: $5
```

**Flow:**
```
TailFile
  ↓
SplitText (line by line)
  ↓
ExtractText
  ↓
RouteOnAttribute
  ${level:equals('ERROR')}
  ↓ true
ExtractText (extract more details from message)
  order.id: .*order (\d+).*
  error.type: .*(timeout|exception|failed).*
  ↓
UpdateRecord (structure the data)
  ↓
PutDatabaseRecord
```

## Advanced Patterns

### Pattern 1: Multi-line Log Aggregation

**Challenge:** Stack traces span multiple lines

**Log Example:**
```
2024-01-15 10:30:45 ERROR Exception occurred
java.lang.NullPointerException: Cannot invoke method
    at com.example.Service.process(Service.java:123)
    at com.example.Controller.handle(Controller.java:45)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
```

**Flow:**
```
TailFile
  ↓
MergeContent
  Merge Strategy: Defragment
  Merge Format: Binary Concatenation
  Correlation Attribute Name: (empty)
  Attribute Strategy: Keep Only Common Attributes
  Minimum Number of Entries: 1
  Maximum Number of Entries: 100
  
  # Demarcator to identify new log entry
  Demarcator Strategy: Text
  Demarcator: ^\d{4}-\d{2}-\d{2}
  
  Max Bin Age: 30 sec
  ↓
ExtractText (extract complete stack trace)
  exception: (?s)(java\..*?Exception.*?)(?=\d{4}-\d{2}-\d{2}|$)
  ↓
[Process complete exception]
```

### Pattern 2: Log Filtering and Sampling

**Reduce log volume by filtering:**

```
TailFile
  ↓
SplitText
  ↓
RouteText
  # Only keep ERROR and WARN
  error_or_warn: (?i)(ERROR|WARN)
  ↓ matched
ExtractText
  ↓
RouteOnAttribute
  # Sample INFO logs (10%)
  ${log.level:equals('INFO')}:and(${random():mod(10):equals(0)})
  ↓ true
[Process sampled INFO logs]
```

### Pattern 3: Real-time Alerting

**Alert on critical errors:**

```
TailFile
  ↓
SplitText
  ↓
RouteText
  critical_error: (?i)(OutOfMemory|StackOverflow|Fatal)
  database_error: (?i)(connection.*timeout|deadlock)
  security_breach: (?i)(unauthorized|authentication.*failed)
  ↓ critical_error
UpdateAttribute
  alert.severity: CRITICAL
  alert.type: ${error.type}
  alert.timestamp: ${now()}
  alert.hostname: ${hostname}
  ↓
InvokeHTTP
  URL: https://alerts.example.com/api/incidents
  HTTP Method: POST
  Content-Type: application/json
  
  Body:
  {
    "severity": "${alert.severity}",
    "type": "${alert.type}",
    "message": "${message}",
    "hostname": "${alert.hostname}",
    "timestamp": "${alert.timestamp}"
  }
  ↓
PutSlack (or PagerDuty, email, etc.)
```

### Pattern 4: Log Aggregation from Multiple Servers

**Collect logs from distributed systems:**

```
Server 1:
TailFile
  ↓
UpdateAttribute
  source.server: server1
  source.environment: production
  ↓
PublishKafka
  Topic: application-logs

Server 2:
TailFile
  ↓
UpdateAttribute
  source.server: server2
  source.environment: production
  ↓
PublishKafka
  Topic: application-logs

Central NiFi:
ConsumeKafka
  Topic: application-logs
  ↓
ExtractGrok
  ↓
PutElasticsearch
  Index: logs-${now():format('yyyy.MM.dd')}
  Index Op: index
  Identifier: ${log.id}
```

### Pattern 5: Log Enrichment

**Add context to logs:**

```
TailFile
  ↓
ExtractText
  user.id: .*user_id=(\d+).*
  transaction.id: .*txn_id=([\w-]+).*
  ↓
InvokeHTTP (get user details)
  URL: http://user-service/api/users/${user.id}
  ↓
EvaluateJsonPath
  user.name: $.name
  user.email: $.email
  user.tier: $.tier
  ↓
MergeContent (merge log + user data)
  ↓
UpdateRecord (add enrichment fields)
  /user/name: ${user.name}
  /user/email: ${user.email}
  /user/tier: ${user.tier}
  ↓
PutElasticsearch
```

### Pattern 6: Log Rotation Handling

**Handle log rotation seamlessly:**

```
TailFile
  File(s) to Tail: /var/log/app/application.log
  Rolling Filename Pattern: application.log.${yyyy-MM-dd}
  State File: ./state/app-tail
  Reread on NUL: false
  
  # When app.log is rotated to app.log.2024-01-15
  # TailFile automatically switches to new app.log
  ↓
UpdateAttribute
  original.file: ${filename}
  rotation.detected: ${tailfile.state:contains('rolled')}
  ↓
RouteOnAttribute
  ${rotation.detected:equals('true')}
  ↓ true
LogAttribute (log rotation event)
```

### Pattern 7: Metrics Extraction from Logs

**Extract performance metrics:**

```
TailFile
  ↓
ExtractText
  Pattern: .*response_time=(\d+)ms.*memory_used=(\d+)MB.*
  response.time: $1
  memory.used: $2
  ↓
UpdateAttribute
  metric.timestamp: ${now()}
  metric.type: application_performance
  ↓
ConvertRecord (to time-series format)
  ↓
PutInfluxDB (or Prometheus)
  
Or:
  ↓
PublishKafka
  Topic: metrics
  ↓
[Stream to Grafana via Kafka]
```

## Log Formats and Parsing

### Common Grok Patterns

**Create patterns file (`/conf/grok-patterns.txt`):**

```grok
# Timestamps
TIMESTAMP_ISO8601 %{YEAR}-%{MONTHNUM}-%{MONTHDAY}[T ]%{HOUR}:?%{MINUTE}(?::?%{SECOND})?%{ISO8601_TIMEZONE}?
APP_TIMESTAMP %{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{HOUR}:%{MINUTE}:%{SECOND}(?:\.%{INT})?

# Log Levels
LOGLEVEL (TRACE|DEBUG|INFO|WARN|WARNING|ERROR|FATAL)

# Common Patterns
JAVACLASS (?:[a-zA-Z$_][a-zA-Z$_0-9]*\.)*[a-zA-Z$_][a-zA-Z$_0-9]*
JAVALOGMESSAGE (.*)

# Application Log
APPLOG %{APP_TIMESTAMP:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:level} %{JAVACLASS:logger} - %{JAVALOGMESSAGE:message}

# Apache Combined Log
COMBINEDAPACHELOG %{COMMONAPACHELOG} %{QS:referrer} %{QS:agent}

# Custom Application
ORDERLOG Order\[id=%{INT:order_id}, customer=%{INT:customer_id}, total=%{NUMBER:total}, status=%{WORD:status}\]
```

### Using ExtractGrok

```
TailFile
  ↓
ExtractGrok
  Grok Expression: %{APPLOG}
  Grok Pattern File: /conf/grok-patterns.txt
  Destination: flowfile-attribute
  Character Set: UTF-8
  Maximum Buffer Size: 1 MB
  ↓
# Extracted attributes:
#   timestamp
#   thread
#   level
#   logger
#   message
```

### Regex vs Grok

**Use Regex (ExtractText) when:**
- Simple patterns
- Custom formats
- Better performance needed

**Use Grok (ExtractGrok) when:**
- Complex patterns
- Standard log formats
- Reusable patterns across flows

## Performance Optimization

### 1. Efficient File Monitoring

```properties
TailFile:
  Lookup Frequency: 10 sec
  # Don't check too frequently
  
  Maximum Age: 24 hours
  # Ignore very old files
  
  Post-Rollover Tail Period: 5 sec
  # How long to read rotated file
```

### 2. Batching Log Lines

```
TailFile
  ↓
MergeContent
  Merge Strategy: Bin-Packing
  Minimum Entries: 100
  Maximum Entries: 1000
  Max Bin Age: 10 sec
  ↓
SplitText (if needed for parsing)
  ↓
[Process batch]
```

### 3. Parallel Processing

```
TailFile (multiple files)
  ↓
DistributeLoad (or HashAttribute)
  Number of Relationships: 5
  ↓ (5 separate lanes)
[Process in parallel]
  ↓
MergeContent (optional - recombine)
```

### 4. Sampling High-Volume Logs

```
TailFile
  ↓
SplitText
  ↓
RouteOnAttribute
  # Keep only 10% of INFO logs
  ${log.level:equals('INFO')}:and(${random():mod(10):equals(0)})
  
  # Keep all ERROR/WARN
  ${log.level:equals('ERROR')}
  ${log.level:equals('WARN')}
```

### 5. Compression

```
TailFile
  ↓
MergeContent (batch logs)
  ↓
CompressContent
  Compression Format: gzip
  Compression Level: 1 (fast)
  ↓
PutHDFS (or PutS3Object)
```

## Real-World Examples

### Example 1: Complete ELK Stack Integration

**Elasticsearch, Logstash, Kibana alternative with NiFi:**

```
TailFile
  File(s): /var/log/app/*.log
  ↓
SplitText (one line per FlowFile)
  ↓
ExtractGrok
  Grok Expression: %{APPLOG}
  ↓
UpdateAttribute
  @timestamp: ${timestamp}
  host: ${hostname}
  application: myapp
  environment: production
  ↓
ConvertRecord
  Record Reader: JsonTreeReader
  Record Writer: JsonRecordSetWriter
  ↓
PutElasticsearch
  Elasticsearch URL: http://localhost:9200
  Index: logs-${now():format('yyyy.MM.dd')}
  Index Operation: index
  Identifier: ${uuid}
  
  # Optional: bulk processing
  Batch Size: 100
```

**Kibana Visualization:**
- Access Kibana: http://localhost:5601
- Create index pattern: `logs-*`
- Build dashboards on log data

### Example 2: Security Log Monitoring

**Monitor auth logs for security events:**

```
TailFile
  File(s): /var/log/auth.log
  ↓
SplitText
  ↓
RouteText
  failed_login: (?i)failed password
  sudo_command: sudo:.*COMMAND
  user_added: useradd
  user_deleted: userdel
  ssh_session: session opened for user
  ↓ failed_login
ExtractText
  ip.address: from ([\d.]+)
  username: for ([\w]+)
  ↓
ExecuteScript (check for brute force)
```

**Groovy Script - Brute Force Detection:**
```groovy
import org.apache.nifi.distributed.cache.client.*

def flowFile = session.get()
if (!flowFile) return

def cacheService = context.getProperty("Cache Service")
    .asControllerService(DistributedMapCacheClient)

def ipAddress = flowFile.getAttribute("ip.address")
def cacheKey = "failed_login_${ipAddress}"

// Get current count
def currentCount = cacheService.get(cacheKey, stringSerializer, stringDeserializer)
def count = currentCount ? currentCount.toInteger() + 1 : 1

// Update count
cacheService.put(cacheKey, count.toString(), stringSerializer, stringSerializer)

flowFile = session.putAttribute(flowFile, "failed.count", count.toString())

// Alert if > 5 failed logins
if (count > 5) {
    flowFile = session.putAttribute(flowFile, "brute.force.detected", "true")
}

session.transfer(flowFile, REL_SUCCESS)
```

```
  ↓
RouteOnAttribute
  ${brute.force.detected:equals('true')}
  ↓ true
UpdateAttribute
  alert.type: BRUTE_FORCE_ATTACK
  alert.severity: HIGH
  ↓
InvokeHTTP (alert security team)
  ↓
PutDatabaseRecord (incident_log table)
```

### Example 3: Application Performance Monitoring

**Extract and visualize performance metrics:**

```
TailFile
  File(s): /var/log/app/performance.log
  ↓
ExtractText
  Pattern: .*endpoint=([\w/]+).*duration=(\d+)ms.*status=(\d+).*
  endpoint: $1
  duration.ms: $2
  status.code: $3
  ↓
UpdateAttribute
  metric.timestamp: ${now()}
  ↓
QueryRecord (calculate percentiles)
  SQL: SELECT 
         endpoint,
         COUNT(*) as request_count,
         AVG(CAST(duration_ms AS DOUBLE)) as avg_duration,
         MAX(CAST(duration_ms AS DOUBLE)) as max_duration,
         MIN(CAST(duration_ms AS DOUBLE)) as min_duration
       FROM FLOWFILE
       GROUP BY endpoint
  ↓
ConvertRecord (JSON → InfluxDB Line Protocol)
  ↓
PutInfluxDB
  Database: application_metrics
  Measurement: api_performance
```

### Example 4: Log Archival and Retention

**Archive logs with retention policy:**

```
TailFile
  ↓
UpdateAttribute
  archive.date: ${now():format('yyyy/MM/dd')}
  ↓
MergeContent
  Merge Strategy: Bin-Packing
  Minimum Entries: 1000
  Maximum Entries: 10000
  Max Bin Age: 5 min
  ↓
CompressContent
  Compression Format: gzip
  ↓
PutHDFS (or PutS3Object)
  Directory: /logs/archive/${archive.date}
  
Cleanup Flow:
GenerateFlowFile (daily at 2 AM)
  ↓
ExecuteScript (delete logs > 90 days)
```

**Python Script - S3 Cleanup:**
```python
import boto3
from datetime import datetime, timedelta

s3 = boto3.client('s3')
bucket = 'my-log-bucket'
retention_days = 90
cutoff_date = datetime.now() - timedelta(days=retention_days)

# List objects
response = s3.list_objects_v2(Bucket=bucket, Prefix='logs/archive/')

deleted_count = 0
for obj in response.get('Contents', []):
    if obj['LastModified'].replace(tzinfo=None) < cutoff_date:
        s3.delete_object(Bucket=bucket, Key=obj['Key'])
        deleted_count += 1

flow_file = session.create()
flow_file = session.putAttribute(flow_file, 'deleted.count', str(deleted_count))
session.transfer(flow_file, REL_SUCCESS)
```

## Monitoring and Debugging

### Monitor TailFile State

**Check state file:**

```bash
# Location specified in State File property
cat ./conf/state/tail-file

# Shows:
# - Files being tailed
# - Current position in each file
# - Last modification times
```

### Debug Flow

```
TailFile
  ↓
LogAttribute
  Log Level: INFO
  Log Payload: true
  Attributes to Log: filename,path,absolute.path,tailfile.*
  ↓
PutFile (debug output)
```

### Performance Metrics

```
ExecuteScript (scheduled every 5 min)
  Language: Groovy
  Script:
```

```groovy
def stats = context.getCounterRepository()
def tailFileStats = stats.getCounters("TailFile")

def flowFile = session.create()
def attributes = [
    'files.tailed': tailFileStats.get("Files Tailed")?.toString(),
    'bytes.read': tailFileStats.get("Bytes Read")?.toString(),
    'lines.read': tailFileStats.get("Lines Read")?.toString()
]

flowFile = session.putAllAttributes(flowFile, attributes)
session.transfer(flowFile, REL_SUCCESS)
```

## Common Issues and Solutions

### Issue 1: Missing Log Lines

**Symptoms:**
- Not all log lines appear in output
- Gaps in data

**Solutions:**

```
1. Check State File location:
   # Ensure directory is writable
   chmod 755 ./conf/state
   
2. Verify file permissions:
   # NiFi user must have read access
   chmod 644 /var/log/app.log
   
3. Check for rotation issues:
   TailFile:
     Rolling Filename Pattern: (must match actual pattern)
     Post-Rollover Tail Period: 30 sec (increase)
```

### Issue 2: Duplicate Log Lines

**Symptoms:**
- Same log lines processed multiple times

**Solutions:**

```
1. Ensure unique State File per TailFile:
   State File: ./state/app1-tail
   # Not shared across processors
   
2. Add deduplication:
   TailFile
     ↓
   DetectDuplicate
     Cache Entry Identifier: ${hash.value}
```

### Issue 3: High Memory Usage

**Symptoms:**
- NiFi OutOfMemoryError
- Slow processing

**Solutions:**

```
1. Batch processing:
   TailFile
     ↓
   MergeContent (batch before processing)
   
2. Reduce buffer size:
   TailFile:
     Maximum Buffer Size: 1 MB (instead of 10 MB)
     
3. Split processing:
   TailFile
     ↓
   SplitText (smaller chunks)
```

### Issue 4: State File Corruption

**Symptoms:**
- `IOException` reading state file
- TailFile not resuming correctly

**Solutions:**

```bash
# Backup state files regularly
cp ./conf/state/tail-file ./conf/state/tail-file.backup

# If corrupted, delete and restart
rm ./conf/state/tail-file
# TailFile will start from beginning or current position
```

### Issue 5: Log Rotation Not Detected

**Symptoms:**
- TailFile stops reading after rotation

**Solutions:**

```properties
TailFile:
  Rolling Filename Pattern: (must exactly match)
  # If logs rotate to: app.log.1, app.log.2
  Rolling Filename Pattern: ${file.name}.${rolling.number}
  
  # For date-based: app-2024-01-15.log
  Rolling Filename Pattern: ${file.name:substringBefore('.log')}-${yyyy-MM-dd}.log
```

## Best Practices

### 1. Use Descriptive State Files

✅ **Good:**
```properties
State File: ./conf/state/application-logs-tail
State File: ./conf/state/nginx-access-tail
```

❌ **Bad:**
```properties
State File: ./conf/state/tail1
State File: ./conf/state/tail2
```

### 2. Handle Multi-line Logs Properly

```
# For stack traces, use MergeContent
TailFile
  ↓
MergeContent (aggregate multi-line)
  Demarcator: ^\d{4}-\d{2}-\d{2}
  ↓
[Process complete log entry]
```

### 3. Implement Backpressure

```
TailFile
  ↓
Connection:
  Back Pressure Object Threshold: 10000
  ↓
[Processing]
# Prevents memory issues with fast log generation
```

### 4. Monitor File Growth

```
ExecuteScript (check log file size)
  ↓
RouteOnAttribute
  ${file.size:toNumber():gt(1000000000)}  # > 1 GB
  ↓ true
InvokeHTTP (alert - large log file)
```

### 5. Regular State Cleanup

```
GenerateFlowFile (weekly)
  ↓
ExecuteScript (cleanup old state entries)
  # Remove state for deleted files
```

### 6. Use Structured Logging

**Application Side (Log4j2):**
```xml
<JsonLayout compact="true" eventEol="true">
    <KeyValuePair key="application" value="myapp"/>
    <KeyValuePair key="environment" value="${env:ENVIRONMENT}"/>
</JsonLayout>
```

**NiFi Side:**
```
TailFile
  ↓
EvaluateJsonPath (easy parsing)
  level: $.level
  message: $.message
  logger: $.loggerName
```

## Summary

Log processing with NiFi enables:
- ✅ Real-time log monitoring with TailFile
- ✅ Flexible parsing with Grok and Regex
- ✅ Multi-line log aggregation
- ✅ Real-time alerting on critical events
- ✅ Log enrichment and correlation
- ✅ Integration with ELK, Splunk, etc.
- ✅ Efficient archival and retention

Key Takeaways:
1. Use TailFile for real-time monitoring
2. Parse logs with ExtractGrok or ExtractText
3. Handle log rotation correctly
4. Implement batching for performance
5. Add alerting for critical errors
6. Archive logs with compression
7. Monitor TailFile state files

```
