## 01_introduction.md
```
```
# Introduction to Apache NiFi

## Overview

Apache NiFi is a powerful data integration and distribution platform that enables automated data flows between systems. It provides a web-based UI for designing, controlling, and monitoring data flows.

## What is Apache NiFi?

NiFi is similar to:
- **Epic Games' Unreal Engine Blueprint System** - Visual programming
- **SQL Server Data Tools (SSDT)** - But with more capabilities

### Key Advantages

- **Universal Connectivity**: Works with virtually any data source/destination
  - Microsoft SQL Server
  - MySQL
  - Oracle
  - Netezza
  - Teradata
  - Elasticsearch
  - Hive
  - HDFS
  - Spark
  - Kafka
  - Kinesis
  - Python
  - Scala

- **Scalability**: Can scale up with available hardware resources
- **Speed**: Processes data as fast as it can be created
- **Web-based Interface**: Accessible through browser
- **Highly Configurable**: Extensive customization options

## Core Concepts

### 1. Flow Files

**Definition**: Objects that move through the NiFi system

Think of flow files as documents being passed between workers:
- Each worker receives the file
- Makes changes or enriches it
- Passes it to the next worker
- Final worker validates and moves it forward

**Flow File Components**:
- **Attributes**: Metadata about the flow file (key-value pairs)
- **Content**: The actual data payload

### 2. Processors

**Definition**: Components that perform operations on flow files

**Types of Operations**:
- **Routing**: Direct flow files to different paths
- **Transformation**: Modify content or attributes
- **Mediation**: Handle data between systems
- **Enrichment**: Add additional information

**Native Processors**: 286+ built-in processors available

### 3. Connections

**Definition**: Queues that link processors together

**Characteristics**:
- Hold flow files temporarily
- Act as buffers between processors
- Configurable backpressure thresholds

### 4. Flow Controller

**Definition**: The scheduler and manager for the entire system

**Responsibilities**:
- Schedules processor execution
- Manages threads
- Allocates resources
- Controls flow file movement

### 5. Process Groups

**Definition**: Containers for organizing related processors

**Benefits**:
- Logical grouping of workflows
- Reusability
- Better organization of complex flows
- Can contain 5-6+ processors working together

## Architecture

```
┌─────────────────────────────────────────┐
│         Web Server (UI)                  │
├─────────────────────────────────────────┤
│         Flow Controller                  │
├─────────────────────────────────────────┤
│         Processors / Extensions          │
├─────────────────────────────────────────┤
│  ┌──────────────┬──────────────────┐   │
│  │ FlowFile     │  Content         │   │
│  │ Repository   │  Repository      │   │
│  └──────────────┴──────────────────┘   │
│  ┌──────────────────────────────────┐  │
│  │   Provenance Repository          │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### Components Explained

1. **Web Server**: Provides the UI interface
2. **Flow Controller**: Handles scheduling and resource management
3. **Processors/Extensions**: Execute data operations
4. **FlowFile Repository**: Tracks flow file state
5. **Content Repository**: Stores flow file content
6. **Provenance Repository**: Maintains data lineage/history

## Data Provenance

**Definition**: Complete tracking of data flow from creation to destination

**Capabilities**:
- Track every operation performed on a flow file
- View complete lineage
- Debug and troubleshoot issues
- Audit compliance

## Installation Options

### Option 1: Docker (Recommended for Testing)

```bash
# Pull the official image
docker pull apache/nifi:latest

# Run NiFi container
docker run --name nifi \
  -p 8443:8443 \
  -d apache/nifi:latest
```

**Note**: Docker setup requires additional configuration for:
- Persistent storage
- Custom drivers
- Network settings

### Option 2: Direct Installation

1. **Download**: Visit https://nifi.apache.org
2. **Extract**: Unzip to desired location
3. **Run**: Execute startup script
   - Windows: `bin/run-nifi.bat`
   - Linux/Mac: `bin/nifi.sh start`

## NiFi User Interface

### Main Canvas

The central workspace where you design flows

**Key Elements**:

1. **Navigation Panel** (Left)
   - Move around large flows
   - Filter processors
   - Zoom controls

2. **Process Group Breadcrumb** (Top Left)
   - Shows current location in flow hierarchy
   - Click to navigate up levels

3. **Configuration Button** (Top Right)
   - Access global settings
   - Enable/disable components
   - Start/stop processors

4. **Component Toolbar** (Top)
   - Processors: Add processing components
   - Input Port: Receive data into process groups
   - Output Port: Send data out of process groups
   - Process Group: Create containers
   - Remote Process Group: Connect to other NiFi instances
   - Funnel: Combine multiple connections
   - Template: Reusable flow patterns
   - Label: Annotate flows

5. **Status Bar** (Bottom Right)
   - Current time (UTC)
   - Active threads count
   - Queued data statistics
   - Running/stopped processor counts
   - Invalid components count
   - Cluster information (if clustered)

### Processor Icons

- **Green Play**: Processor is running
- **Red Square**: Processor is stopped
- **Warning**: Invalid configuration
- **Disabled**: Processor is disabled

### Queue Metrics

Between processors, you'll see:
- **Number**: Flow files in queue
- **Size**: Total data size in queue

## Creating Your First Flow

### Basic Steps

1. **Add Processor**: Drag from toolbar
2. **Configure**: Double-click to set properties
3. **Connect**: Click center dot, drag to next processor
4. **Validate**: Check for warning icons
5. **Start**: Right-click processor → Start

### Example: Simple File Movement

```
GetFile → PutFile
```

This basic flow:
- Reads files from a directory (GetFile)
- Writes them to another directory (PutFile)

## Best Practices

### Naming Conventions

- **Rename Processors**: Use descriptive names
  - Bad: "GetFile"
  - Good: "Get Customer CSV Files"

### Flow Organization

- Use Process Groups for logical separation
- Add Labels to document complex flows
- Keep canvas organized and readable

### Scheduling

- Don't run everything at 0 seconds (continuous)
- Set appropriate intervals based on needs
- Consider resource impact

### Error Handling

- Always handle failure relationships
- Log errors appropriately
- Implement retry logic when appropriate

## Common Use Cases

1. **Data Integration**: Move data between systems
2. **ETL Operations**: Extract, Transform, Load workflows
3. **Real-time Processing**: Stream processing
4. **IoT Data Collection**: Sensor data aggregation
5. **Log Aggregation**: Centralize log collection
6. **API Integration**: Connect to REST/SOAP services

## Performance Considerations

### Thread Management

- Configure concurrent tasks per processor
- Monitor active threads
- Adjust based on system resources

### Backpressure

- Set appropriate queue thresholds
- Prevent memory issues
- Control flow rate

### Clustering

- Distribute workload across nodes
- Use Primary Node execution when needed
- Leverage Remote Process Groups

## Next Steps

1. Complete the [First Flow Tutorial](02_first_flow.md)
2. Learn about [Data Transformation](03_jolt_transformation.md)
3. Explore [Database Integration](09_database_connections.md)

## Additional Resources

- Official Documentation: https://nifi.apache.org/docs.html
- Expression Language Guide: https://nifi.apache.org/docs/nifi-docs/html/expression-language-guide.html
- Component Documentation: Available in UI (right-click → View Usage)

## Glossary

- **FlowFile**: Data object moving through NiFi
- **Processor**: Component that acts on flow files
- **Connection**: Queue between processors
- **Relationship**: Named output from a processor (success, failure, etc.)
- **Attribute**: Metadata key-value pair on a flow file
- **Content**: The actual data payload of a flow file
- **Provenance**: Historical record of flow file processing
```
