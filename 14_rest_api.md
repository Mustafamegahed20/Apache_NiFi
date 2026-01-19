## 14_rest_api.md

```markdown
```
# NiFi REST API Guide

## Overview

The Apache NiFi REST API provides programmatic access to all NiFi functionality, enabling automation, monitoring, and integration with external systems. This guide covers authentication, common operations, automation patterns, and best practices.

## API Basics

### Base URL Structure

```
http://[nifi-host]:[port]/nifi-api
```

**Examples:**
```
http://localhost:8080/nifi-api
https://nifi.example.com:8443/nifi-api
```

### API Documentation

**Swagger UI:**
```
http://localhost:8080/nifi-docs/rest-api/index.html
```

### Common Endpoints

| Endpoint | Purpose |
|----------|---------|
| `/flow/process-groups/{id}` | Get process group details |
| `/process-groups/{id}/processors` | Manage processors |
| `/controller-services` | Manage controller services |
| `/flow/connections/{id}` | Manage connections |
| `/snippets` | Create/move/copy components |
| `/provenance` | Query data provenance |
| `/system-diagnostics` | System health metrics |
| `/controller/cluster` | Cluster information |

## Authentication

### Single User (Development)

**No authentication required for HTTP:**

```bash
curl http://localhost:8080/nifi-api/flow/about
```

### Token-Based Authentication (Production)

**1. Get Access Token:**

```bash
# Request token
curl -X POST https://nifi.example.com:8443/nifi-api/access/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=yourpassword"
```

**Response:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhZG1pbiIsImlhdCI6MTYxNjI...
```

**2. Use Token in Requests:**

```bash
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl https://nifi.example.com:8443/nifi-api/flow/about \
  -H "Authorization: Bearer $TOKEN"
```

### Certificate-Based Authentication

**With client certificate:**

```bash
curl https://nifi.example.com:8443/nifi-api/flow/about \
  --cert client.crt \
  --key client.key \
  --cacert ca.crt
```

## Common API Operations

### Get System Information

**About NiFi:**

```bash
curl http://localhost:8080/nifi-api/flow/about
```

**Response:**
```json
{
  "about": {
    "title": "NiFi",
    "version": "1.24.0",
    "uri": "http://localhost:8080/nifi-api",
    "timezone": "UTC",
    "contentViewerUrl": "/nifi-content-viewer/",
    "buildTag": "nifi-1.24.0",
    "buildRevision": "abc123"
  }
}
```

**Cluster Status:**

```bash
curl http://localhost:8080/nifi-api/controller/cluster
```

**System Diagnostics:**

```bash
curl http://localhost:8080/nifi-api/system-diagnostics
```

**Response:**
```json
{
  "systemDiagnostics": {
    "aggregateSnapshot": {
      "totalNonHeap": "256 MB",
      "totalHeap": "4 GB",
      "usedHeap": "1.5 GB",
      "maxHeap": "4 GB",
      "heapUtilization": "37%",
      "availableProcessors": 8,
      "processorLoadAverage": 2.5,
      "totalThreads": 150,
      "daemonThreads": 120
    }
  }
}
```

### Process Groups

**Get Root Process Group:**

```bash
# Get root PG ID first
ROOT_PG_ID=$(curl -s http://localhost:8080/nifi-api/flow/process-groups/root | jq -r '.processGroupFlow.id')

echo "Root Process Group ID: $ROOT_PG_ID"
```

**Get Process Group Details:**

```bash
curl http://localhost:8080/nifi-api/flow/process-groups/{pg-id}
```

**Create Process Group:**

```bash
curl -X POST http://localhost:8080/nifi-api/process-groups/{parent-pg-id}/process-groups \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": 0
    },
    "component": {
      "name": "My Process Group",
      "position": {
        "x": 100,
        "y": 100
      }
    }
  }'
```

**Response:**
```json
{
  "revision": {
    "version": 1
  },
  "id": "abc-123-def-456",
  "component": {
    "id": "abc-123-def-456",
    "name": "My Process Group",
    "position": {
      "x": 100.0,
      "y": 100.0
    }
  }
}
```

### Processors

**List Processors in Process Group:**

```bash
curl http://localhost:8080/nifi-api/process-groups/{pg-id}/processors
```

**Get Specific Processor:**

```bash
curl http://localhost:8080/nifi-api/processors/{processor-id}
```

**Create Processor:**

```bash
curl -X POST http://localhost:8080/nifi-api/process-groups/{pg-id}/processors \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": 0
    },
    "component": {
      "type": "org.apache.nifi.processors.standard.GetFile",
      "name": "GetFile-SourceData",
      "position": {
        "x": 200,
        "y": 200
      },
      "config": {
        "properties": {
          "Input Directory": "/data/input",
          "Keep Source File": "false",
          "File Filter": ".*\\.csv"
        },
        "schedulingPeriod": "10 sec",
        "schedulingStrategy": "TIMER_DRIVEN",
        "concurrentlySchedulableTaskCount": "1"
      }
    }
  }'
```

**Update Processor:**

```bash
# First get current version
PROCESSOR_INFO=$(curl -s http://localhost:8080/nifi-api/processors/{processor-id})
VERSION=$(echo $PROCESSOR_INFO | jq -r '.revision.version')

# Update processor
curl -X PUT http://localhost:8080/nifi-api/processors/{processor-id} \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": '$VERSION'
    },
    "component": {
      "id": "'{processor-id}'",
      "config": {
        "properties": {
          "Input Directory": "/data/new-input"
        }
      }
    }
  }'
```

**Start/Stop Processor:**

```bash
# Get current version first
VERSION=$(curl -s http://localhost:8080/nifi-api/processors/{processor-id} | jq -r '.revision.version')

# Start processor
curl -X PUT http://localhost:8080/nifi-api/processors/{processor-id}/run-status \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": '$VERSION'
    },
    "state": "RUNNING"
  }'

# Stop processor
curl -X PUT http://localhost:8080/nifi-api/processors/{processor-id}/run-status \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": '$VERSION'
    },
    "state": "STOPPED"
  }'
```

**Delete Processor:**

```bash
VERSION=$(curl -s http://localhost:8080/nifi-api/processors/{processor-id} | jq -r '.revision.version')

curl -X DELETE "http://localhost:8080/nifi-api/processors/{processor-id}?version=$VERSION"
```

### Connections

**Create Connection:**

```bash
curl -X POST http://localhost:8080/nifi-api/process-groups/{pg-id}/connections \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": 0
    },
    "component": {
      "source": {
        "id": "{source-processor-id}",
        "type": "PROCESSOR"
      },
      "destination": {
        "id": "{destination-processor-id}",
        "type": "PROCESSOR"
      },
      "selectedRelationships": ["success"],
      "backPressureDataSizeThreshold": "1 GB",
      "backPressureObjectThreshold": "10000",
      "flowFileExpiration": "0 sec",
      "prioritizers": []
    }
  }'
```

**List Connections:**

```bash
curl http://localhost:8080/nifi-api/process-groups/{pg-id}/connections
```

**Delete FlowFiles from Connection:**

```bash
# Create drop request
DROP_REQUEST=$(curl -X POST http://localhost:8080/nifi-api/flowfile-queues/{connection-id}/drop-requests \
  -H "Content-Type: application/json" \
  -d '{}')

DROP_REQUEST_ID=$(echo $DROP_REQUEST | jq -r '.dropRequest.id')

# Poll until complete
while true; do
  STATUS=$(curl -s http://localhost:8080/nifi-api/flowfile-queues/{connection-id}/drop-requests/$DROP_REQUEST_ID \
    | jq -r '.dropRequest.finished')
  
  if [ "$STATUS" = "true" ]; then
    echo "Drop complete"
    break
  fi
  
  echo "Dropping FlowFiles..."
  sleep 2
done
```

### Controller Services

**List Controller Services:**

```bash
curl http://localhost:8080/nifi-api/flow/controller-services
```

**Create Controller Service:**

```bash
curl -X POST http://localhost:8080/nifi-api/process-groups/{pg-id}/controller-services \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": 0
    },
    "component": {
      "type": "org.apache.nifi.dbcp.DBCPConnectionPool",
      "name": "MySQL-ConnectionPool",
      "properties": {
        "Database Connection URL": "jdbc:mysql://localhost:3306/mydb",
        "Database Driver Class Name": "com.mysql.cj.jdbc.Driver",
        "Database Driver Location(s)": "/opt/nifi/jdbc/mysql-connector.jar",
        "Database User": "user",
        "Password": "password"
      }
    }
  }'
```

**Enable Controller Service:**

```bash
VERSION=$(curl -s http://localhost:8080/nifi-api/controller-services/{service-id} | jq -r '.revision.version')

curl -X PUT http://localhost:8080/nifi-api/controller-services/{service-id}/run-status \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": '$VERSION'
    },
    "state": "ENABLED"
  }'
```

### Variables

**Get Process Group Variables:**

```bash
curl http://localhost:8080/nifi-api/process-groups/{pg-id}/variable-registry
```

**Update Variables:**

```bash
# Get current variables
CURRENT=$(curl -s http://localhost:8080/nifi-api/process-groups/{pg-id}/variable-registry)
VERSION=$(echo $CURRENT | jq -r '.processGroupRevision.version')

# Update
curl -X PUT http://localhost:8080/nifi-api/process-groups/{pg-id}/variable-registry \
  -H "Content-Type: application/json" \
  -d '{
    "processGroupRevision": {
      "version": '$VERSION'
    },
    "variableRegistry": {
      "variables": [
        {
          "variable": {
            "name": "db.host",
            "value": "new-db-host.example.com"
          }
        },
        {
          "variable": {
            "name": "api.url",
            "value": "https://api.example.com"
          }
        }
      ]
    }
  }'
```

## Bulk Operations

### Start All Processors in Process Group

```bash
#!/bin/bash
# start-all-processors.sh

PG_ID="$1"

# Get all processors
PROCESSORS=$(curl -s http://localhost:8080/nifi-api/process-groups/$PG_ID/processors \
  | jq -r '.processors[].id')

for PROC_ID in $PROCESSORS; do
  echo "Starting processor: $PROC_ID"
  
  VERSION=$(curl -s http://localhost:8080/nifi-api/processors/$PROC_ID | jq -r '.revision.version')
  
  curl -X PUT http://localhost:8080/nifi-api/processors/$PROC_ID/run-status \
    -H "Content-Type: application/json" \
    -d '{
      "revision": {"version": '$VERSION'},
      "state": "RUNNING"
    }' > /dev/null
  
  echo "Started: $PROC_ID"
done

echo "All processors started"
```

### Stop All Processors

```bash
#!/bin/bash
# stop-all-processors.sh

PG_ID="$1"

PROCESSORS=$(curl -s http://localhost:8080/nifi-api/process-groups/$PG_ID/processors \
  | jq -r '.processors[].id')

for PROC_ID in $PROCESSORS; do
  echo "Stopping processor: $PROC_ID"
  
  VERSION=$(curl -s http://localhost:8080/nifi-api/processors/$PROC_ID | jq -r '.revision.version')
  
  curl -X PUT http://localhost:8080/nifi-api/processors/$PROC_ID/run-status \
    -H "Content-Type: application/json" \
    -d '{
      "revision": {"version": '$VERSION'},
      "state": "STOPPED"
    }' > /dev/null
done

echo "All processors stopped"
```

### Empty All Queues

```bash
#!/bin/bash
# empty-all-queues.sh

PG_ID="$1"

# Get all connections
CONNECTIONS=$(curl -s http://localhost:8080/nifi-api/process-groups/$PG_ID/connections \
  | jq -r '.connections[].id')

for CONN_ID in $CONNECTIONS; do
  echo "Emptying queue: $CONN_ID"
  
  # Create drop request
  DROP_REQ=$(curl -X POST http://localhost:8080/nifi-api/flowfile-queues/$CONN_ID/drop-requests \
    -H "Content-Type: application/json" \
    -d '{}')
  
  DROP_REQ_ID=$(echo $DROP_REQ | jq -r '.dropRequest.id')
  
  # Wait for completion
  while true; do
    STATUS=$(curl -s http://localhost:8080/nifi-api/flowfile-queues/$CONN_ID/drop-requests/$DROP_REQ_ID \
      | jq -r '.dropRequest.finished')
    
    if [ "$STATUS" = "true" ]; then
      break
    fi
    sleep 1
  done
  
  echo "Emptied: $CONN_ID"
done

echo "All queues emptied"
```

## Template Management

### Upload Template

```bash
curl -X POST http://localhost:8080/nifi-api/process-groups/{pg-id}/templates/upload \
  -F template=@my-template.xml
```

### Download Template

```bash
curl http://localhost:8080/nifi-api/templates/{template-id}/download \
  -o downloaded-template.xml
```

### Instantiate Template

```bash
curl -X POST http://localhost:8080/nifi-api/process-groups/{pg-id}/template-instance \
  -H "Content-Type: application/json" \
  -d '{
    "templateId": "{template-id}",
    "originX": 100,
    "originY": 100
  }'
```

## Monitoring and Metrics

### Get Processor Statistics

```bash
curl http://localhost:8080/nifi-api/flow/processors/{processor-id}/status
```

**Response:**
```json
{
  "processorStatus": {
    "id": "abc-123",
    "name": "GetFile",
    "runStatus": "Running",
    "statsLastRefreshed": "10:30:00 UTC",
    "aggregateSnapshot": {
      "bytesRead": 1048576,
      "bytesWritten": 1048576,
      "read": "1 MB",
      "written": "1 MB",
      "flowFilesIn": 100,
      "flowFilesOut": 100,
      "bytesIn": 1048576,
      "bytesOut": 1048576,
      "input": "1 MB",
      "output": "1 MB",
      "taskCount": 100,
      "tasksDurationNanos": 5000000000,
      "tasks": "100",
      "tasksDuration": "00:00:05.000"
    }
  }
}
```

### Get Connection Status

```bash
curl http://localhost:8080/nifi-api/flow/connections/{connection-id}/status
```

**Response:**
```json
{
  "connectionStatus": {
    "id": "def-456",
    "name": "success",
    "statsLastRefreshed": "10:30:00 UTC",
    "aggregateSnapshot": {
      "flowFilesIn": 1000,
      "bytesIn": 10485760,
      "input": "10 MB",
      "flowFilesOut": 1000,
      "bytesOut": 10485760,
      "output": "10 MB",
      "flowFilesQueued": 50,
      "bytesQueued": 524288,
      "queued": "512 KB",
      "queuedSize": "512 KB",
      "queuedCount": "50"
    }
  }
}
```

### Get Process Group Status

```bash
curl http://localhost:8080/nifi-api/flow/process-groups/{pg-id}/status
```

### Real-Time Monitoring Script

```python
#!/usr/bin/env python3
# monitor-nifi.py

import requests
import time
import json

NIFI_URL = "http://localhost:8080/nifi-api"
PG_ID = "root"  # or specific PG ID

def get_pg_status():
    response = requests.get(f"{NIFI_URL}/flow/process-groups/{PG_ID}/status")
    return response.json()

def monitor():
    while True:
        status = get_pg_status()
        stats = status['processGroupStatus']['aggregateSnapshot']
        
        print(f"\n--- NiFi Status at {time.strftime('%H:%M:%S')} ---")
        print(f"FlowFiles Queued: {stats['queuedCount']}")
        print(f"Bytes Queued: {stats['queued']}")
        print(f"Active Threads: {stats['activeThreadCount']}")
        print(f"FlowFiles In (5 min): {stats['flowFilesIn']}")
        print(f"FlowFiles Out (5 min): {stats['flowFilesOut']}")
        print(f"Bytes Read: {stats['read']}")
        print(f"Bytes Written: {stats['written']}")
        
        # Check for issues
        if int(stats['queuedCount'].replace(',', '')) > 10000:
            print("⚠️  WARNING: High queue count!")
        
        time.sleep(30)

if __name__ == "__main__":
    monitor()
```

## Provenance

### Query Provenance

```bash
curl -X POST http://localhost:8080/nifi-api/provenance \
  -H "Content-Type: application/json" \
  -d '{
    "request": {
      "maxResults": 100,
      "searchTerms": {
        "ProcessorID": "{processor-id}"
      },
      "startDate": "01/15/2024 00:00:00 UTC",
      "endDate": "01/15/2024 23:59:59 UTC"
    }
  }'
```

**Response:**
```json
{
  "provenance": {
    "id": "query-123",
    "uri": "http://localhost:8080/nifi-api/provenance/query-123",
    "finished": true,
    "percentCompleted": 100,
    "results": {
      "provenanceEvents": [
        {
          "eventId": 12345,
          "eventTime": "01/15/2024 10:30:00.000 UTC",
          "eventType": "RECEIVE",
          "componentId": "abc-123",
          "componentName": "GetFile",
          "attributes": {
            "filename": "data.csv",
            "path": "/input/"
          },
          "fileSize": "1024 bytes"
        }
      ],
      "totalCount": 1,
      "total": "1"
    }
  }
}
```

### Get Provenance Event

```bash
curl http://localhost:8080/nifi-api/provenance/events/{event-id}
```

### Replay FlowFile

```bash
curl -X POST http://localhost:8080/nifi-api/provenance-events/{event-id}/replays \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Flow Registry Integration

### List Registry Clients

```bash
curl http://localhost:8080/nifi-api/flow/registries
```

### Create Registry Client

```bash
curl -X POST http://localhost:8080/nifi-api/controller/registry-clients \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {
      "version": 0
    },
    "component": {
      "name": "Production Registry",
      "uri": "http://registry.example.com:18080",
      "description": "Production flow registry"
    }
  }'
```

### Version Control - Start

```bash
curl -X POST http://localhost:8080/nifi-api/versions/process-groups/{pg-id} \
  -H "Content-Type: application/json" \
  -d '{
    "processGroupRevision": {
      "version": 0
    },
    "versionedFlow": {
      "registryId": "{registry-id}",
      "bucketId": "{bucket-id}",
      "flowName": "My Flow",
      "flowDescription": "Production data flow",
      "comments": "Initial commit"
    }
  }'
```

### Version Control - Commit

```bash
curl -X PUT http://localhost:8080/nifi-api/versions/process-groups/{pg-id} \
  -H "Content-Type: application/json" \
  -d '{
    "processGroupRevision": {
      "version": {current-version}
    },
    "versionControlInformation": {
      "registryId": "{registry-id}",
      "bucketId": "{bucket-id}",
      "flowId": "{flow-id}",
      "version": {new-version}
    },
    "comments": "Updated error handling"
  }'
```

### Version Control - Update

```bash
curl -X POST http://localhost:8080/nifi-api/versions/update-requests/process-groups/{pg-id} \
  -H "Content-Type: application/json" \
  -d '{
    "processGroupRevision": {
      "version": {current-version}
    },
    "versionControlInformation": {
      "registryId": "{registry-id}",
      "bucketId": "{bucket-id}",
      "flowId": "{flow-id}",
      "version": {target-version}
    }
  }'
```

## Automation Patterns

### Health Check Script

```python
#!/usr/bin/env python3
# nifi-health-check.py

import requests
import sys

NIFI_URL = "http://localhost:8080/nifi-api"

def check_nifi_health():
    try:
        # Check if NiFi is responding
        response = requests.get(f"{NIFI_URL}/flow/about", timeout=10)
        if response.status_code != 200:
            print("❌ NiFi is not responding properly")
            return False
        
        # Check cluster status (if clustered)
        cluster_response = requests.get(f"{NIFI_URL}/controller/cluster")
        if cluster_response.status_code == 200:
            cluster_data = cluster_response.json()
            if 'cluster' in cluster_data:
                nodes = cluster_data['cluster']['nodes']
                disconnected = [n for n in nodes if n['status'] != 'CONNECTED']
                if disconnected:
                    print(f"❌ {len(disconnected)} nodes disconnected")
                    return False
        
        # Check system diagnostics
        diag_response = requests.get(f"{NIFI_URL}/system-diagnostics")
        diag = diag_response.json()['systemDiagnostics']['aggregateSnapshot']
        
        heap_util = float(diag['heapUtilization'].replace('%', ''))
        if heap_util > 90:
            print(f"⚠️  High heap usage: {heap_util}%")
            return False
        
        print("✅ NiFi is healthy")
        return True
        
    except Exception as e:
        print(f"❌ Health check failed: {e}")
        return False

if __name__ == "__main__":
    healthy = check_nifi_health()
    sys.exit(0 if healthy else 1)
```

### Auto-Restart Failed Processors

```python
#!/usr/bin/env python3
# auto-restart-failed.py

import requests
import time

NIFI_URL = "http://localhost:8080/nifi-api"
PG_ID = "root"

def get_failed_processors():
    response = requests.get(f"{NIFI_URL}/process-groups/{PG_ID}/processors")
    processors = response.json()['processors']
    
    failed = []
    for proc in processors:
        status = proc['status']['aggregateSnapshot']
        if status['runStatus'] == 'Stopped' and int(status.get('activeThreadCount', 0)) == 0:
            # Check if it has errors
            if int(status.get('bytesIn', 0)) > 0 and int(status.get('bytesOut', 0)) == 0:
                failed.append(proc)
    
    return failed

def restart_processor(proc_id):
    # Get current version
    proc_resp = requests.get(f"{NIFI_URL}/processors/{proc_id}")
    version = proc_resp.json()['revision']['version']
    
    # Start processor
    requests.put(
        f"{NIFI_URL}/processors/{proc_id}/run-status",
        json={
            "revision": {"version": version},
            "state": "RUNNING"
        }
    )
    print(f"Restarted processor: {proc_id}")

def main():
    while True:
        failed = get_failed_processors()
        
        for proc in failed:
            print(f"Found failed processor: {proc['component']['name']}")
            restart_processor(proc['id'])
        
        time.sleep(60)  # Check every minute

if __name__ == "__main__":
    main()
```

### Deploy Flow Script

```bash
#!/bin/bash
# deploy-flow.sh

NIFI_URL="http://localhost:8080/nifi-api"
REGISTRY_ID="your-registry-id"
BUCKET_ID="your-bucket-id"
FLOW_ID="your-flow-id"
TARGET_PG_ID="root"

echo "Deploying flow from registry..."

# Create process group for flow
PG_RESPONSE=$(curl -s -X POST $NIFI_URL/process-groups/$TARGET_PG_ID/process-groups \
  -H "Content-Type: application/json" \
  -d '{
    "revision": {"version": 0},
    "component": {
      "name": "Deployed Flow",
      "position": {"x": 100, "y": 100}
    }
  }')

NEW_PG_ID=$(echo $PG_RESPONSE | jq -r '.id')
echo "Created process group: $NEW_PG_ID"

# Import flow from registry
curl -s -X POST $NIFI_URL/versions/process-groups/$NEW_PG_ID \
  -H "Content-Type: application/json" \
  -d '{
    "processGroupRevision": {"version": 0},
    "versionedFlow": {
      "registryId": "'$REGISTRY_ID'",
      "bucketId": "'$BUCKET_ID'",
      "flowId": "'$FLOW_ID'",
      "version": 1
    }
  }' > /dev/null

echo "Imported flow version 1"

# Start all processors
echo "Starting processors..."
./start-all-processors.sh $NEW_PG_ID

echo "Deployment complete!"
```

### Backup Script

```bash
#!/bin/bash
# backup-nifi.sh

NIFI_URL="http://localhost:8080/nifi-api"
BACKUP_DIR="/backup/nifi/$(date +%Y%m%d_%H%M%S)"

mkdir -p $BACKUP_DIR

echo "Backing up NiFi configuration..."

# Get all templates
curl -s $NIFI_URL/flow/templates | jq -r '.templates[].template.uri' | while read TEMPLATE_URI; do
  TEMPLATE_ID=$(basename $TEMPLATE_URI)
  TEMPLATE_NAME=$(curl -s $NIFI_URL/templates/$TEMPLATE_ID | jq -r '.template.name')
  
  echo "Backing up template: $TEMPLATE_NAME"
  curl -s $NIFI_URL/templates/$TEMPLATE_ID/download \
    > "$BACKUP_DIR/template-$TEMPLATE_NAME.xml"
done

# Backup flow configuration
echo "Backing up flow configuration..."
curl -s $NIFI_URL/flow/process-groups/root \
  > "$BACKUP_DIR/flow-config.json"

# Backup controller services
echo "Backing up controller services..."
curl -s $NIFI_URL/flow/controller-services \
  > "$BACKUP_DIR/controller-services.json"

# Backup reporting tasks
echo "Backing up reporting tasks..."
curl -s $NIFI_URL/flow/reporting-tasks \
  > "$BACKUP_DIR/reporting-tasks.json"

echo "Backup complete: $BACKUP_DIR"
```

## Python Library - nipyapi

### Installation

```bash
pip install nipyapi
```

### Basic Usage

```python
import nipyapi

# Connect to NiFi
nipyapi.config.nifi_config.host = 'http://localhost:8080/nifi-api'

# Get root process group
root_pg = nipyapi.canvas.get_root_pg_id()

# List all processors
processors = nipyapi.canvas.list_all_processors()
for proc in processors:
    print(f"{proc.component.name}: {proc.status.aggregate_snapshot.run_status}")

# Create a processor
new_proc = nipyapi.canvas.create_processor(
    parent_pg=root_pg,
    processor=nipyapi.canvas.get_processor_type('GetFile'),
    location=(100.0, 100.0),
    name='GetFile-Source'
)

# Configure processor
nipyapi.canvas.update_processor(
    processor=new_proc,
    update={
        'properties': {
            'Input Directory': '/data/input',
            'Keep Source File': 'false'
        },
        'schedulingPeriod': '10 sec'
    }
)

# Start processor
nipyapi.canvas.schedule_processor(new_proc, True)

# Create connection
source_proc = processors[0]
dest_proc = processors[1]

connection = nipyapi.canvas.create_connection(
    source=source_proc,
    target=dest_proc,
    relationships=['success']
)
```

### Advanced Operations

```python
# Deploy from registry
nipyapi.versioning.deploy_flow_version(
    parent_id=root_pg,
    location=(200.0, 200.0),
    bucket_id='bucket-id',
    flow_id='flow-id',
    version=1,
    registry_client_id='registry-client-id'
)

# Get flow status
status = nipyapi.canvas.get_process_group_status(root_pg)
print(f"Queued FlowFiles: {status.aggregate_snapshot.queued_count}")
print(f"Queued Bytes: {status.aggregate_snapshot.queued}")

# Stop all processors
for proc in processors:
    nipyapi.canvas.schedule_processor(proc, False)

# Empty all queues
connections = nipyapi.canvas.list_all_connections()
for conn in connections:
    nipyapi.canvas.purge_connection(conn.id)
```

## Best Practices

### 1. Always Check Versions

```bash
# Get current version before update
VERSION=$(curl -s http://localhost:8080/nifi-api/processors/{id} \
  | jq -r '.revision.version')

# Use in update
curl -X PUT http://localhost:8080/nifi-api/processors/{id} \
  -d '{"revision": {"version": '$VERSION'}, ...}'
```

### 2. Handle Errors Gracefully

```python
import requests

def safe_api_call(url, method='GET', **kwargs):
    try:
        response = requests.request(method, url, **kwargs)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.HTTPError as e:
        print(f"HTTP Error: {e.response.status_code}")
        print(f"Response: {e.response.text}")
        return None
    except Exception as e:
        print(f"Error: {e}")
        return None
```

### 3. Use Pagination for Large Results

```python
def get_all_provenance_events(search_terms):
    all_events = []
    offset = 0
    limit = 1000
    
    while True:
        response = requests.post(
            f"{NIFI_URL}/provenance",
            json={
                "request": {
                    "maxResults": limit,
                    "offset": offset,
                    "searchTerms": search_terms
                }
            }
        )
        
        data = response.json()
        events = data['provenance']['results']['provenanceEvents']
        all_events.extend(events)
        
        if len(events) < limit:
            break
        
        offset += limit
    
    return all_events
```

### 4. Implement Retry Logic

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=5):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

@retry(max_attempts=3, delay=5)
def start_processor(processor_id):
    # API call here
    pass
```

### 5. Use Environment Variables

```bash
#!/bin/bash

# Configuration
export NIFI_URL="${NIFI_URL:-http://localhost:8080/nifi-api}"
export NIFI_USER="${NIFI_USER:-admin}"
export NIFI_PASS="${NIFI_PASS:-}"

# Get token if credentials provided
if [ -n "$NIFI_PASS" ]; then
    export NIFI_TOKEN=$(curl -s -X POST $NIFI_URL/access/token \
        -H "Content-Type: application/x-www-form-urlencoded" \
        -d "username=$NIFI_USER&password=$NIFI_PASS")
fi

# Use token in requests
curl $NIFI_URL/flow/about \
    -H "Authorization: Bearer $NIFI_TOKEN"
```

## Common Issues and Solutions

### Issue 1: Version Conflict

**Error:** `Version mismatch. Expected X, got Y`

**Solution:**
```bash
# Always fetch latest version before update
LATEST=$(curl -s http://localhost:8080/nifi-api/processors/{id} \
  | jq -r '.revision.version')

# Use latest version in update
```

### Issue 2: 409 Conflict

**Error:** `409 Conflict - Component is running`

**Solution:**
```bash
# Stop processor first
curl -X PUT http://localhost:8080/nifi-api/processors/{id}/run-status \
  -d '{"revision": {"version": X}, "state": "STOPPED"}'

# Wait for it to stop
sleep 5

# Then update/delete
```

### Issue 3: Authentication Timeout

**Error:** `401 Unauthorized`

**Solution:**
```bash
# Refresh token periodically
get_new_token() {
    TOKEN=$(curl -s -X POST $NIFI_URL/access/token \
        -d "username=$USER&password=$PASS")
    echo $TOKEN
}

# Call before long-running operations
TOKEN=$(get_new_token)
```

## Summary

The NiFi REST API enables:
- ✅ Full automation of NiFi operations
- ✅ Programmatic flow deployment
- ✅ Real-time monitoring and metrics
- ✅ Integration with CI/CD pipelines
- ✅ Bulk operations across flows
- ✅ Custom tooling and dashboards

Key Takeaways:
1. Always check component versions before updates
2. Use proper authentication in production
3. Handle errors gracefully with retries
4. Leverage libraries like nipyapi for Python
5. Implement health checks and monitoring
6. Automate deployments with scripts
7. Use versioning for flow management
8. Monitor performance metrics regularly

## Resources

- **API Documentation:** http://localhost:8080/nifi-docs/rest-api/
- **nipyapi GitHub:** https://github.com/Chaffelson/nipyapi
- **NiFi Source Code:** https://github.com/apache/nifi
- **Apache NiFi Website:** https://nifi.apache.org
```

---

Perfect! I've completed all 14 comprehensive Apache NiFi guides:

1. ✅ Introduction to Apache NiFi
2. ✅ First NiFi Flow - CSV to MySQL
3. ✅ Data Transformation with Jolt
4. ✅ Creating CSV Files from Flows
5. ✅ Twitter Data Integration
6. ✅ Query Record Processor
7. ✅ WebSocket Integration
8. ✅ XML Processors
9. ✅ Database Connections
10. ✅ Wait/Notify Pattern
11. ✅ Kafka Integration
12. ✅ TailFile and Log Processing
13. ✅ Development Environment Setup
14. ✅ NiFi REST API

Each guide includes:
- Clear explanations and concepts
- Configuration examples
- Real-world use cases
- Code samples and scripts
- Best practices
- Troubleshooting tips
- Complete working examples

These guides should provide a comprehensive resource for learning and working with Apache NiFi from beginner to advanced levels!
