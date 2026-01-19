## 13_dev_environment.md

```markdown
```
# Development Environment Setup for Apache NiFi

## Overview

This guide covers setting up a professional NiFi development environment, including local installation, version control, testing strategies, CI/CD integration, and best practices for flow development.

## Local Development Setup

### Installation Options

#### Option 1: Binary Installation

**Download and Install:**

```bash
# Download NiFi
cd /opt
wget https://dlcdn.apache.org/nifi/1.24.0/nifi-1.24.0-bin.tar.gz

# Extract
tar -xzf nifi-1.24.0-bin.tar.gz
cd nifi-1.24.0

# Start NiFi
./bin/nifi.sh start

# Check status
./bin/nifi.sh status

# Access UI
# https://localhost:8443/nifi
```

**Initial Setup:**

```bash
# Get auto-generated credentials (first time)
cat ./conf/nifi.properties | grep "username"
cat ./conf/nifi.properties | grep "password"

# Or set custom credentials
./bin/nifi.sh set-single-user-credentials admin YourSecurePassword123
```

#### Option 2: Docker Installation

**Docker Compose Setup:**

```yaml
# docker-compose.yml
version: '3'
services:
  nifi:
    image: apache/nifi:1.24.0
    container_name: nifi
    ports:
      - "8443:8443"     # HTTPS UI
      - "8080:8080"     # HTTP UI (if enabled)
      - "10000:10000"   # ListenHTTP
    environment:
      - NIFI_WEB_HTTPS_PORT=8443
      - NIFI_WEB_HTTPS_HOST=0.0.0.0
      - SINGLE_USER_CREDENTIALS_USERNAME=admin
      - SINGLE_USER_CREDENTIALS_PASSWORD=adminadminadmin
      - NIFI_SENSITIVE_PROPS_KEY=your-secret-key-here
    volumes:
      - ./nifi-conf:/opt/nifi/nifi-current/conf
      - ./nifi-database:/opt/nifi/nifi-current/database_repository
      - ./nifi-flowfile:/opt/nifi/nifi-current/flowfile_repository
      - ./nifi-content:/opt/nifi/nifi-current/content_repository
      - ./nifi-provenance:/opt/nifi/nifi-current/provenance_repository
      - ./nifi-state:/opt/nifi/nifi-current/state
      - ./nifi-logs:/opt/nifi/nifi-current/logs
      - ./nifi-jdbc:/opt/nifi/nifi-current/jdbc
    networks:
      - nifi-net

  # Optional: Add Registry for version control
  registry:
    image: apache/nifi-registry:1.24.0
    container_name: nifi-registry
    ports:
      - "18080:18080"
    environment:
      - NIFI_REGISTRY_WEB_HTTP_PORT=18080
    volumes:
      - ./registry-database:/opt/nifi-registry/nifi-registry-current/database
      - ./registry-flow-storage:/opt/nifi-registry/nifi-registry-current/flow_storage
    networks:
      - nifi-net

networks:
  nifi-net:
    driver: bridge
```

**Start:**

```bash
# Create directories
mkdir -p nifi-conf nifi-database nifi-flowfile nifi-content \
         nifi-provenance nifi-state nifi-logs nifi-jdbc \
         registry-database registry-flow-storage

# Start services
docker-compose up -d

# Check logs
docker-compose logs -f nifi

# Access NiFi
# https://localhost:8443/nifi
# Username: admin
# Password: adminadminadmin
```

#### Option 3: Kubernetes Deployment

**Helm Chart Installation:**

```bash
# Add NiFi Helm repository
helm repo add cetic https://cetic.github.io/helm-charts
helm repo update

# Create values file
cat > nifi-values.yaml <<EOF
replicaCount: 1

image:
  repository: apache/nifi
  tag: 1.24.0

properties:
  webProxyHost: nifi.example.com
  clusterSecure: true

persistence:
  enabled: true
  storageClass: standard
  size: 10Gi

resources:
  requests:
    memory: "2Gi"
    cpu: "1000m"
  limits:
    memory: "4Gi"
    cpu: "2000m"

auth:
  singleUser:
    username: admin
    password: changeme123

ingress:
  enabled: true
  hosts:
    - host: nifi.example.com
      paths:
        - path: /
EOF

# Install
helm install nifi cetic/nifi -f nifi-values.yaml

# Check status
kubectl get pods -l app=nifi
```

## Configuration

### Essential nifi.properties Settings

**Development Environment:**

```properties
# Core Properties
nifi.version=1.24.0
nifi.flow.configuration.file=./conf/flow.xml.gz
nifi.flow.configuration.archive.enabled=true
nifi.flow.configuration.archive.dir=./conf/archive/

# Web Properties - Development (HTTP)
nifi.web.http.host=localhost
nifi.web.http.port=8080
nifi.web.https.host=
nifi.web.https.port=

# Security - Disabled for dev
nifi.security.user.login.identity.provider=
nifi.security.needClientAuth=false

# State Management
nifi.state.management.configuration.file=./conf/state-management.xml
nifi.state.management.embedded.zookeeper.start=false

# Repository Settings - Smaller for dev
nifi.content.repository.implementation=org.apache.nifi.controller.repository.FileSystemRepository
nifi.content.claim.max.appendable.size=1 MB
nifi.content.claim.max.flow.files=100
nifi.content.repository.archive.max.retention.period=1 hour
nifi.content.repository.archive.max.usage.percentage=50%

# FlowFile Repository
nifi.flowfile.repository.implementation=org.apache.nifi.controller.repository.WriteAheadFlowFileRepository
nifi.flowfile.repository.checkpoint.interval=2 mins
nifi.swap.manager.implementation=org.apache.nifi.controller.FileSystemSwapManager
nifi.queue.swap.threshold=20000

# Provenance Repository - Smaller for dev
nifi.provenance.repository.max.storage.time=24 hours
nifi.provenance.repository.max.storage.size=1 GB

# Component Status Repository
nifi.components.status.repository.buffer.size=1440
nifi.components.status.snapshot.frequency=1 min

# Performance
nifi.nar.library.autoload.directory=./extensions
nifi.nar.working.directory=./work/nar/
```

**Production-Like Environment:**

```properties
# Web Properties - Production (HTTPS)
nifi.web.http.host=
nifi.web.http.port=
nifi.web.https.host=0.0.0.0
nifi.web.https.port=8443

# Security - Enabled
nifi.security.keystore=/opt/nifi/conf/keystore.jks
nifi.security.keystoreType=JKS
nifi.security.keystorePasswd=
nifi.security.keyPasswd=
nifi.security.truststore=/opt/nifi/conf/truststore.jks
nifi.security.truststoreType=JKS
nifi.security.truststorePasswd=

# Cluster Configuration
nifi.cluster.is.node=true
nifi.cluster.node.address=nifi-node-1
nifi.cluster.node.protocol.port=11443
nifi.cluster.flow.election.max.wait.time=5 mins
nifi.cluster.flow.election.max.candidates=3

# ZooKeeper
nifi.zookeeper.connect.string=zk1:2181,zk2:2181,zk3:2181

# Larger repositories
nifi.content.repository.archive.max.retention.period=7 days
nifi.provenance.repository.max.storage.time=30 days
nifi.provenance.repository.max.storage.size=10 GB
```

### JVM Configuration

**bootstrap.conf:**

```properties
# Development
java.arg.2=-Xms1g
java.arg.3=-Xmx2g
java.arg.14=-XX:+UseG1GC
java.arg.15=-XX:MaxGCPauseMillis=500

# Production
java.arg.2=-Xms4g
java.arg.3=-Xmx8g
java.arg.14=-XX:+UseG1GC
java.arg.15=-XX:MaxGCPauseMillis=200
java.arg.16=-XX:+HeapDumpOnOutOfMemoryError
java.arg.17=-XX:HeapDumpPath=./logs/
```

### Logging Configuration

**logback.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="30 seconds">
    <!-- Console appender for development -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>

    <!-- File appender -->
    <appender name="APP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>./logs/nifi-app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>./logs/nifi-app_%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Development: INFO to console and file -->
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="APP_FILE" />
    </root>

    <!-- Debug specific packages -->
    <logger name="org.apache.nifi.web.api" level="DEBUG"/>
    <logger name="org.apache.nifi.processors" level="DEBUG"/>
</configuration>
```

## Version Control with NiFi Registry

### NiFi Registry Setup

**1. Install NiFi Registry:**

```bash
# Download
wget https://dlcdn.apache.org/nifi/nifi-registry/1.24.0/nifi-registry-1.24.0-bin.tar.gz

# Extract
tar -xzf nifi-registry-1.24.0-bin.tar.gz
cd nifi-registry-1.24.0

# Start
./bin/nifi-registry.sh start

# Access
# http://localhost:18080/nifi-registry
```

**2. Configure Flow Persistence Provider:**

**providers.xml:**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<providers>
    <!-- Git Flow Persistence Provider -->
    <flowPersistenceProvider>
        <class>org.apache.nifi.registry.provider.flow.git.GitFlowPersistenceProvider</class>
        <property name="Flow Storage Directory">./flow_storage</property>
        <property name="Remote To Push">origin</property>
        <property name="Remote Access User">git-user</property>
        <property name="Remote Access Password"></property>
    </flowPersistenceProvider>
    
    <!-- Or File System Provider -->
    <flowPersistenceProvider>
        <class>org.apache.nifi.registry.provider.flow.FileSystemFlowPersistenceProvider</class>
        <property name="Flow Storage Directory">./flow_storage</property>
    </flowPersistenceProvider>
</providers>
```

### Integrating NiFi with Registry

**1. Add Registry Client in NiFi:**

```
Settings (top-right gear icon)
  → Management Controller Services
  → Registry Clients tab
  → Add Registry Client
  
Name: Dev Registry
URL: http://localhost:18080
Description: Local development registry
```

**2. Create Bucket in Registry:**

```
Registry UI → Buckets
  → New Bucket
  
Bucket Name: development-flows
Description: Development environment flows
```

**3. Version Control a Process Group:**

```
Right-click Process Group
  → Version → Start version control
  
Registry: Dev Registry
Bucket: development-flows
Flow Name: my-data-flow
Flow Description: Main data processing flow
Comments: Initial commit
```

**4. Commit Changes:**

```
Right-click Process Group (with green checkmark)
  → Version → Commit local changes
  
Comments: Added error handling logic
```

**5. Update from Registry:**

```
Right-click Process Group (with red asterisk)
  → Version → Change version
  
Select version to deploy
  → Change
```

### Git Integration

**Setup Git Backend:**

```bash
# Initialize Git repository for flows
cd /opt/nifi-registry/flow_storage
git init
git remote add origin https://github.com/your-org/nifi-flows.git

# Configure Git
git config user.name "NiFi Registry"
git config user.email "nifi@example.com"

# Push to remote
git push -u origin master
```

**Directory Structure:**

```
flow_storage/
├── development-flows/
│   ├── my-data-flow/
│   │   ├── 1.snapshot
│   │   ├── 2.snapshot
│   │   └── bucket.yml
│   └── bucket.yml
└── production-flows/
    └── bucket.yml
```

## Flow Development Best Practices

### Project Structure

**Organized Process Groups:**

```
Root Canvas
├── [PG] 01 - Ingestion
│   ├── GetFile
│   ├── ListenHTTP
│   └── ConsumeKafka
├── [PG] 02 - Validation
│   ├── ValidateRecord
│   └── RouteOnAttribute
├── [PG] 03 - Transformation
│   ├── JoltTransformJSON
│   ├── QueryRecord
│   └── UpdateRecord
├── [PG] 04 - Enrichment
│   ├── InvokeHTTP
│   └── MergeContent
└── [PG] 05 - Output
    ├── PutDatabaseRecord
    ├── PutElasticsearch
    └── PutKafka
```

### Naming Conventions

**Process Groups:**

```
✅ Good:
  01-Ingestion-CustomerData
  02-Validation-SchemaCheck
  03-Transform-Normalize
  
❌ Bad:
  ProcessGroup1
  Data
  Transform
```

**Processors:**

```
✅ Good:
  GetFile-SourceOrders
  ValidateRecord-OrderSchema
  PutSQL-InsertOrders
  
❌ Bad:
  GetFile
  Validate
  Put
```

**Controller Services:**

```
✅ Good:
  DBCPConnectionPool-MySQLProduction
  JsonTreeReader-OrderSchema
  DistributedMapCache-SessionState
  
❌ Bad:
  ConnectionPool1
  Reader
  Cache
```

**Variables:**

```
✅ Good:
  db.url
  api.endpoint
  kafka.brokers
  batch.size
  
❌ Bad:
  var1
  url
  endpoint
```

### Documentation Standards

**Process Group Documentation:**

```
Description:
Purpose: Ingests customer order data from multiple sources
Inputs: CSV files, Kafka topic, REST API
Outputs: Validated JSON to processing queue
Dependencies: MySQL database, Kafka cluster
Owner: Data Team
Last Updated: 2024-01-15
```

**Processor Comments:**

```
Processor Comment:
Validates order records against schema v2.1
Rejection criteria:
  - Missing required fields (order_id, customer_id)
  - Invalid date formats
  - Negative amounts
Invalid records → quarantine queue
```

**Connection Labels:**

```
✅ Use labels to explain routing:
  "Valid Records"
  "Schema Errors"
  "Enriched Data"
  "Failed Enrichment - Retry"
```

### Variable Management

**Process Group Variables:**

```
Development Variables:
  db.host: localhost
  db.port: 3306
  db.name: dev_orders
  db.user: dev_user
  db.password: dev_pass
  
  api.url: http://localhost:8080
  kafka.brokers: localhost:9092
  
  batch.size: 100
  retry.max: 3

Production Variables (override):
  db.host: prod-db.example.com
  db.name: prod_orders
  db.user: prod_user
  
  api.url: https://api.example.com
  kafka.brokers: kafka1:9092,kafka2:9092,kafka3:9092
  
  batch.size: 1000
```

**Usage in Processors:**

```properties
ExecuteSQL:
  Database Connection URL: jdbc:mysql://${db.host}:${db.port}/${db.name}
  Database User: ${db.user}
  Password: ${db.password}
  
InvokeHTTP:
  Remote URL: ${api.url}/customers
  
PublishKafka:
  Kafka Brokers: ${kafka.brokers}
```

## Testing Strategies

### Unit Testing

**Test Individual Processors:**

```
Test Flow:
GenerateFlowFile
  Custom Text: {"id": 1, "name": "Test"}
  ↓
[Processor Under Test]
  ↓
PutFile (test-output/)
  ↓
ExecuteScript (validation)
```

**Validation Script (Python):**

```python
import json

flow_file = session.get()
if not flow_file:
    return

# Read content
content = None
def read_content(input_stream, output_stream):
    global content
    content = IOUtils.toString(input_stream, StandardCharsets.UTF_8)
    output_stream.write(content.encode('utf-8'))

session.read(flow_file, read_content)

# Validate
data = json.loads(content)
assert 'id' in data, "Missing id field"
assert 'name' in data, "Missing name field"
assert isinstance(data['id'], int), "ID must be integer"

# Pass validation
flow_file = session.putAttribute(flow_file, 'validation.passed', 'true')
session.transfer(flow_file, REL_SUCCESS)
```

### Integration Testing

**Test Complete Flow:**

```
Test Setup Process Group:
GenerateFlowFile
  ↓
ReplaceText (create test data)
  Replacement Value:
  [
    {"order_id": 1, "customer_id": 100, "total": 50.00},
    {"order_id": 2, "customer_id": 101, "total": 75.50},
    {"order_id": 3, "customer_id": 100, "total": 125.00}
  ]
  ↓
PutFile (test-input/)

Main Flow:
GetFile (test-input/)
  ↓
[Complete Processing Pipeline]
  ↓
PutFile (test-output/)

Validation Process Group:
GetFile (test-output/)
  ↓
ExecuteScript (validate results)
  ↓
LogAttribute (test results)
```

### Automated Testing with NiFi CLI

**Install NiFi CLI:**

```bash
# NiFi Toolkit includes CLI
wget https://dlcdn.apache.org/nifi/1.24.0/nifi-toolkit-1.24.0-bin.tar.gz
tar -xzf nifi-toolkit-1.24.0-bin.tar.gz
cd nifi-toolkit-1.24.0

# Setup CLI
./bin/cli.sh session set nifi.url http://localhost:8080
```

**Test Script:**

```bash
#!/bin/bash
# test-flow.sh

# Import test flow
FLOW_ID=$(./bin/cli.sh nifi pg-import \
  -u http://localhost:8080 \
  -i root \
  -b test-flows \
  -f test-order-processing \
  -o json | jq -r '.id')

echo "Imported flow: $FLOW_ID"

# Start processors
./bin/cli.sh nifi pg-start -u http://localhost:8080 -pgid $FLOW_ID

# Wait for processing
sleep 10

# Check results
OUTPUT_COUNT=$(./bin/cli.sh nifi pg-status \
  -u http://localhost:8080 \
  -pgid $FLOW_ID \
  -o json | jq '.aggregateSnapshot.flowFilesOut')

if [ "$OUTPUT_COUNT" -eq "3" ]; then
    echo "Test PASSED: Processed 3 records"
    exit 0
else
    echo "Test FAILED: Expected 3, got $OUTPUT_COUNT"
    exit 1
fi
```

### Load Testing

**Generate High Volume:**

```
GenerateFlowFile
  Run Schedule: 0 sec (continuous)
  Concurrent Tasks: 10
  Custom Text: {"id": ${random():mod(10000)}}
  ↓
UpdateAttribute
  test.timestamp: ${now()}
  ↓
[Flow Under Test]
  ↓
UpdateAttribute
  process.duration: ${now():minus(${test.timestamp})}
  ↓
LogAttribute (performance metrics)
```

**Monitor Performance:**

```
ExecuteScript (every 30 sec)
  Language: Groovy
  Script:
```

```groovy
def stats = context.getCounterRepository()
def processorStats = stats.getCounters("FlowUnderTest")

def throughput = processorStats.get("FlowFiles Out") / 30 // per second
def avgDuration = processorStats.get("Processing Duration") / processorStats.get("FlowFiles Out")

def flowFile = session.create()
flowFile = session.putAttribute(flowFile, "throughput.fps", throughput.toString())
flowFile = session.putAttribute(flowFile, "avg.duration.ms", avgDuration.toString())

if (throughput < 100) {
    flowFile = session.putAttribute(flowFile, "performance.alert", "LOW_THROUGHPUT")
}

session.transfer(flowFile, REL_SUCCESS)
```

## CI/CD Integration

### GitHub Actions

**.github/workflows/nifi-deploy.yml:**

```yaml
name: Deploy NiFi Flow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup NiFi CLI
        run: |
          wget https://dlcdn.apache.org/nifi/1.24.0/nifi-toolkit-1.24.0-bin.tar.gz
          tar -xzf nifi-toolkit-1.24.0-bin.tar.gz
          echo "$PWD/nifi-toolkit-1.24.0/bin" >> $GITHUB_PATH
      
      - name: Validate Flow Definition
        run: |
          # Validate JSON structure
          jq empty flows/*.json
          
      - name: Check Flow Version
        run: |
          # Ensure version incremented
          NEW_VERSION=$(jq -r '.versionedFlowSnapshot.snapshotMetadata.version' flows/main-flow.json)
          echo "Flow version: $NEW_VERSION"

  deploy-dev:
    needs: validate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Dev
        env:
          NIFI_URL: ${{ secrets.DEV_NIFI_URL }}
          NIFI_USER: ${{ secrets.NIFI_USER }}
          NIFI_PASS: ${{ secrets.NIFI_PASS }}
        run: |
          # Import flow to dev environment
          cli.sh session set nifi.url $NIFI_URL
          cli.sh nifi pg-import \
            -u $NIFI_URL \
            -i root \
            -b development-flows \
            -f main-data-flow
      
      - name: Run Integration Tests
        run: |
          ./scripts/test-flow.sh

  deploy-prod:
    needs: deploy-dev
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Production
        env:
          NIFI_URL: ${{ secrets.PROD_NIFI_URL }}
        run: |
          cli.sh nifi pg-import \
            -u $NIFI_URL \
            -i root \
            -b production-flows \
            -f main-data-flow
      
      - name: Smoke Test
        run: |
          ./scripts/smoke-test.sh
```

### Jenkins Pipeline

**Jenkinsfile:**

```groovy
pipeline {
    agent any
    
    environment {
        NIFI_REGISTRY = 'http://registry.example.com:18080'
        DEV_NIFI = 'http://dev-nifi.example.com:8080'
        PROD_NIFI = 'http://prod-nifi.example.com:8080'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/your-org/nifi-flows.git'
            }
        }
        
        stage('Validate') {
            steps {
                script {
                    sh '''
                        # Validate flow JSON
                        for file in flows/*.json; do
                            jq empty "$file" || exit 1
                        done
                    '''
                }
            }
        }
        
        stage('Deploy to Dev') {
            steps {
                script {
                    sh '''
                        ./nifi-toolkit/bin/cli.sh session set nifi.url $DEV_NIFI
                        ./nifi-toolkit/bin/cli.sh nifi pg-import \
                            -u $DEV_NIFI \
                            -i root \
                            -b development-flows \
                            -f main-flow
                    '''
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    sh './scripts/run-tests.sh'
                }
            }
        }
        
        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
                
                script {
                    sh '''
                        ./nifi-toolkit/bin/cli.sh session set nifi.url $PROD_NIFI
                        ./nifi-toolkit/bin/cli.sh nifi pg-import \
                            -u $PROD_NIFI \
                            -i root \
                            -b production-flows \
                            -f main-flow
                    '''
                }
            }
        }
        
        stage('Verify') {
            steps {
                script {
                    sh './scripts/verify-deployment.sh'
                }
            }
        }
    }
    
    post {
        success {
            slackSend color: 'good', 
                      message: "NiFi deployment succeeded: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: 'danger', 
                      message: "NiFi deployment failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
```

## Deployment Strategies

### Blue-Green Deployment

**Setup:**

```
Blue Environment (Current Production):
  - NiFi Cluster: nifi-blue-1, nifi-blue-2, nifi-blue-3
  - Processing: main-flow v1.0

Green Environment (New Version):
  - NiFi Cluster: nifi-green-1, nifi-green-2, nifi-green-3
  - Processing: main-flow v2.0 (testing)

Load Balancer:
  - Routes traffic to Blue
  - Switch to Green when validated
```

**Deployment Script:**

```bash
#!/bin/bash
# blue-green-deploy.sh

GREEN_ENV="http://nifi-green-lb:8080"
BLUE_ENV="http://nifi-blue-lb:8080"

# Deploy to green
echo "Deploying to green environment..."
cli.sh nifi pg-import -u $GREEN_ENV -b production-flows -f main-flow

# Verify green
echo "Verifying green environment..."
./scripts/verify-deployment.sh $GREEN_ENV

if [ $? -eq 0 ]; then
    echo "Green verified. Switching traffic..."
    # Update load balancer
    curl -X POST http://lb-api/switch-to-green
    
    echo "Monitoring for 10 minutes..."
    sleep 600
    
    # Check for errors
    ERROR_COUNT=$(curl -s http://monitoring/api/errors?env=green&duration=10m | jq '.count')
    
    if [ "$ERROR_COUNT" -lt "10" ]; then
        echo "Deployment successful. Green is now production."
    else
        echo "Errors detected. Rolling back to blue..."
        curl -X POST http://lb-api/switch-to-blue
    fi
else
    echo "Green verification failed. Keeping blue as production."
    exit 1
fi
```

### Canary Deployment

**Route 10% traffic to new version:**

```
Load Balancer Configuration:
  - 90% → NiFi v1.0
  - 10% → NiFi v2.0 (canary)

Monitor canary for:
  - Error rates
  - Latency
  - Throughput

If successful:
  - Increase to 25%
  - Then 50%
  - Then 100%
```

### Rolling Update

**For clustered NiFi:**

```bash
#!/bin/bash
# rolling-update.sh

NODES=("nifi-1" "nifi-2" "nifi-3")

for node in "${NODES[@]}"; do
    echo "Updating $node..."
    
    # Disconnect node from cluster
    ssh $node "sudo systemctl stop nifi"
    
    # Deploy new flow
    cli.sh nifi pg-import -u http://$node:8080 -b prod -f main-flow
    
    # Start node
    ssh $node "sudo systemctl start nifi"
    
    # Wait for node to rejoin
    sleep 60
    
    # Verify node health
    STATUS=$(curl -s http://$node:8080/nifi-api/controller/cluster | jq -r '.cluster.nodes[] | select(.address=="'$node'") | .status')
    
    if [ "$STATUS" != "CONNECTED" ]; then
        echo "Node $node failed to reconnect! Aborting."
        exit 1
    fi
    
    echo "$node updated successfully"
done

echo "Rolling update complete"
```

## Environment Management

### Multi-Environment Setup

**Directory Structure:**

```
nifi-environments/
├── dev/
│   ├── docker-compose.yml
│   ├── nifi.properties
│   └── variables.json
├── test/
│   ├── docker-compose.yml
│   ├── nifi.properties
│   └── variables.json
├── staging/
│   ├── kubernetes/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   └── variables.json
└── production/
    ├── kubernetes/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── configmap.yaml
    └── variables.json
```

**Environment Variables (dev):**

```json
{
  "variables": {
    "db.host": "localhost",
    "db.port": "3306",
    "db.name": "dev_db",
    "api.url": "http://localhost:8000",
    "kafka.brokers": "localhost:9092",
    "log.level": "DEBUG"
  }
}
```

**Environment Variables (production):**

```json
{
  "variables": {
    "db.host": "prod-db-cluster.example.com",
    "db.port": "3306",
    "db.name": "production_db",
    "api.url": "https://api.example.com",
    "kafka.brokers": "kafka-1:9092,kafka-2:9092,kafka-3:9092",
    "log.level": "INFO"
  }
}
```

### Configuration Management

**Use Parameter Contexts:**

```
NiFi UI → Parameter Contexts
  → Add Parameter Context

Name: Database-Config
Description: Database connection parameters

Parameters:
  db.host: prod-db.example.com (sensitive: false)
  db.port: 3306 (sensitive: false)
  db.user: app_user (sensitive: false)
  db.password: ********* (sensitive: true)
```

**Apply to Process Group:**

```
Right-click Process Group
  → Configure
  → Settings tab
  → Parameter Context: Database-Config
```

## Monitoring and Debugging

### Enable Debug Logging

**For specific processor:**

```
logback.xml:
<logger name="org.apache.nifi.processors.standard.GetFile" level="DEBUG"/>
<logger name="org.apache.nifi.processors.standard.PutSQL" level="DEBUG"/>
```

### Debug Flow

```
[Processor Under Debug]
  ↓ (all relationships)
LogAttribute
  Log Level: INFO
  Log Payload: true
  Attributes to Log: (regex) .*
  ↓
PutFile (debug-output/)
```

### Remote Debugging

**Enable in bootstrap.conf:**

```properties
java.arg.debug=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000
```

**Connect from IDE:**

```
IntelliJ IDEA:
  Run → Edit Configurations
  → Add New → Remote JVM Debug
  
  Host: localhost
  Port: 8000
  Command line args: -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000
```

## Summary

Development environment best practices:
- ✅ Use Docker for consistent local setup
- ✅ Implement version control with Registry
- ✅ Follow naming and documentation standards
- ✅ Use variables for environment-specific config
- ✅ Implement automated testing
- ✅ Set up CI/CD pipelines
- ✅ Use proper deployment strategies
- ✅ Monitor and debug effectively

Key Takeaways:
1. Start with local Docker setup
2. Use NiFi Registry for version control
3. Organize flows with Process Groups
4. Document everything clearly
5. Test thoroughly before production
6. Automate deployments with CI/CD
7. Use parameter contexts for configuration
8. Monitor performance continuously

## Next Steps

- Learn [NiFi REST API](14_rest_api.md)
- Review [TailFile for Logs](12_tailfile_logs.md)
- Explore [Kafka Integration](11_kafka_integration.md)
```
