# Apache_NiFi
# Apache NiFi Comprehensive Guide

I'll create detailed markdown guides for each topic covered in the video transcripts. Let me organize this into comprehensive documentation.

## Table of Contents

1. [Introduction to Apache NiFi](01_introduction.md)
2. [First NiFi Flow - CSV to MySQL](02_first_flow.md)
3. [Data Transformation with Jolt](03_jolt_transformation.md)
4. [Creating CSV Files from Flows](04_creating_csv_files.md)
5. [Twitter Data Integration](05_twitter_integration.md)
6. [Query Record Processor](06_query_record.md)
7. [WebSocket Integration](07_websocket.md)
8. [XML Processors](08_xml_processors.md)
9. [Database Connections](09_database_connections.md)
10. [Wait/Notify Pattern](10_wait_notify.md)
11. [Kafka Integration](11_kafka_integration.md)
12. [TailFile and Log Processing](12_tailfile_logs.md)
13. [Development Environment Setup](13_dev_environment.md)
14. [NiFi REST API](14_rest_api.md)

---

Let me create each guide:

## 01_introduction.md

```markdown
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

## 02_first_flow.md

```markdown
# Creating Your First NiFi Flow: CSV to MySQL

## Overview

This guide walks through creating a complete data flow that:
1. Reads CSV files from a directory
2. Splits files into individual records
3. Transforms data to JSON
4. Loads data into MySQL database

## Prerequisites

- NiFi installed and running
- MySQL database accessible
- Sample CSV data files
- MySQL JDBC driver

## The Data

### Source: USGS Earthquake Data

**Location**: CSV files in a directory

**Sample Structure**:
```csv
time,latitude,longitude,depth,mag,magType,nst,gap,dmin,rms,net,id,updated,place,type,horizontalError,depthError,magError,magNst,status,locationSource,magSource
2019-12-01T00:00:00.000Z,36.1234,-117.5678,2.3,1.2,ml,12,45,0.01234,0.12,ci,ci12345678,2019-12-01T00:05:00.000Z,"9km SE of Ridgecrest, CA",earthquake,0.5,1.2,0.1,5,automatic,ci,ci
```

**Data Characteristics**:
- Multiple CSV files (one per month)
- ~300,000+ total records
- Varying file sizes
- Contains header row

## Flow Architecture

```
GetFile → SplitText → ConvertRecord → ConvertJSONToSQL → PutSQL
```

## Step-by-Step Implementation

### Step 1: Create Process Group

**Why?** Organize related processors together

1. Click **Process Group** icon in toolbar
2. Draw on canvas
3. Name it: `USGS CSV Data`
4. Double-click to enter group

### Step 2: Get Files (GetFile Processor)

**Purpose**: Read CSV files from directory

#### Configuration

1. **Add Processor**:
   ```
   Search: "GetFile"
   Add to canvas
   ```

2. **Rename**:
   - Name: `Get USGS Files`

3. **Settings Tab**:
   - Bulletin Level: `WARN` (default is fine)
   - Automatically Terminate Relationships: None yet

4. **Scheduling Tab**:
   ```
   Scheduling Strategy: Timer driven
   Run Schedule: 5 sec (for testing, increase for production)
   Concurrent Tasks: 1
   Execution: All nodes (or Primary Node for single instance)
   ```

5. **Properties Tab**:
   ```properties
   Input Directory: /path/to/csv/files
   File Filter: .*\.csv (regex for .csv files)
   Path Filter: (leave empty)
   Recurse Subdirectories: false
   Keep Source File: true (for testing, false for production)
   Minimum File Age: 0 sec
   Maximum File Age: (leave empty)
   Minimum File Size: 0 B
   Maximum File Size: (leave empty)
   Ignore Hidden Files: true
   Polling Interval: 0 sec
   Batch Size: 1 (for testing)
   ```

**Important Properties Explained**:

- **Input Directory**: Absolute path to source files
- **Keep Source File**: 
  - `true`: File remains after processing (good for testing)
  - `false`: File is deleted after processing (good for production)
- **Batch Size**: Number of files to process per execution

6. **Apply** and close

#### Testing GetFile

1. Right-click processor → **Start**
2. Check queue after execution (should see flow file count)
3. Right-click processor → **Stop**
4. Right-click queue → **List queue**
5. Click **Info** icon on a flow file
6. View **Attributes** and **Content** tabs

**Expected Attributes**:
```
absolute.path: /path/to/csv/files
filename: 2019-12.csv
path: ./
file.size: 622340 (bytes)
file.owner: user
file.group: group
file.permissions: rw-r--r--
file.lastModifiedTime: 2024-01-15T10:30:00
```

**Expected Content**: Full CSV file with all rows

### Step 3: Split Text (SplitText Processor)

**Purpose**: Convert one flow file (entire CSV) into multiple flow files (one per row)

#### Why Split?

- MySQL insert requires individual records
- Better error handling per row
- Parallel processing capability

#### Configuration

1. **Add Processor**: Search `SplitText`
2. **Rename**: `Split Text USGS`
3. **Connect**: GetFile → SplitText (success relationship)

4. **Properties**:
   ```properties
   Line Split Count: 1 (one row per flow file)
   Maximum Fragment Size: 0 (not using)
   Header Line Count: 1 (CSV has header)
   Header Line Marker Characters: (leave empty)
   Remove Trailing Newlines: true
   ```

**Key Property - Header Line Count**:
- `1`: Skip first line (header) from being included in data splits
- Each split will still get the header prepended to it

5. **Settings Tab**:
   ```
   Automatically Terminate Relationships:
   ☑ failure
   ☑ original
   ```

#### Testing Split

1. Start SplitText processor
2. Observe: If source file has 3,455 rows → Creates 3,454 flow files
3. List queue and view content
4. Each flow file should have:
   ```
   Line 1: Header row
   Line 2: Data row
   ```

### Step 4: Convert Record (ConvertRecord Processor)

**Purpose**: Convert CSV text to JSON format

#### Why JSON?

- MySQL integration requires JSON format
- StandardJSON format required for ConvertJSONtoSQL processor
- Enables downstream transformations

#### Controller Services Setup

**IMPORTANT**: Before configuring ConvertRecord, set up Controller Services

##### A. Create CSV Reader

1. Click canvas background
2. Click **Configuration** (gear icon)
3. Select **Controller Services** tab
4. Click **+** to add service
5. Search: `CSV Reader`
6. Add: `CSVReader`

7. **Configure CSV Reader**:
   ```
   Click gear icon on CSVReader
   
   Properties:
   - Schema Access Strategy: Use String Fields From Header
   - Treat First Line as Header: true
   - Schema Registry: (leave empty)
   - Schema Name: (leave empty)
   - Schema Version: (leave empty)
   - Schema Branch: (leave empty)
   - CSV Format: Apache Commons CSV
   - Value Separator: ,
   - Skip Header Line: false (already handled by SplitText)
   - Quote Character: "
   - Escape Character: \
   - Comment Marker: (none)
   - Null String: (empty)
   - Trim Fields: true
   - Date Format: (auto-detect)
   - Time Format: (auto-detect)
   - Timestamp Format: (auto-detect)
   ```

8. Click **Apply**
9. Click **Enable** (lightning bolt icon)

##### B. Create JSON Writer

1. Still in Controller Services tab
2. Click **+** to add service
3. Search: `JSON`
4. Add: `JsonRecordSetWriter`

5. **Configure JSON Writer**:
   ```
   Properties:
   - Schema Write Strategy: Do Not Write Schema
   - Schema Access Strategy: Inherit Record Schema
   - Schema Registry: (leave empty)
   - Schema Name: (leave empty)
   - Schema Version: (leave empty)
   - Schema Branch: (leave empty)
   - Output Grouping: One Line Per Object (important!)
   - Compression Format: none
   - Pretty Print JSON: false
   - Suppress Null Values: Never Suppress
   - Include Record Path: false
   ```

**Critical Setting**:
- **Output Grouping**: `One Line Per Object`
  - NOT array format
  - Each record is separate JSON object
  - Required for ConvertJSONtoSQL

6. Click **Apply**
7. Click **Enable**

##### Exit Controller Services

1. Click **Apply** on configuration dialog
2. Close configuration
3. Return to canvas

#### Configure ConvertRecord Processor

1. **Add Processor**: Search `ConvertRecord`
2. **Rename**: `Convert Record Text to JSON`
3. **Connect**: SplitText (splits) → ConvertRecord

4. **Properties**:
   ```properties
   Record Reader: CSVReader (select from dropdown)
   Record Writer: JsonRecordSetWriter (select from dropdown)
   Include Zero Record FlowFiles: false
   ```

5. **Settings Tab**:
   ```
   Automatically Terminate Relationships:
   ☑ failure
   ```

#### Testing Convert

1. Empty queues from previous testing
2. Start GetFile → Start SplitText → Start ConvertRecord
3. Check output queue
4. List queue and view flow file content

**Expected Output**:
```json
{"time":"2019-12-01T00:00:00.000Z","latitude":"36.1234","longitude":"-117.5678","depth":"2.3","mag":"1.2","magType":"ml","nst":"12","gap":"45","dmin":"0.01234","rms":"0.12","net":"ci","id":"ci12345678","updated":"2019-12-01T00:05:00.000Z","place":"9km SE of Ridgecrest, CA","type":"earthquake"}
```

### Step 5: Convert JSON to SQL (ConvertJSONtoSQL Processor)

**Purpose**: Generate SQL INSERT statements from JSON

#### Database Connection Setup

##### Create MySQL Connection Pool

1. **From Processor Configuration**:
   - Add ConvertJSONtoSQL processor
   - In Properties tab, next to "JDBC Connection Pool"
   - Click **Create new service**
   - Or go to root canvas → Configuration → Controller Services

2. **Add Service**:
   ```
   Search: DBCPConnectionPool
   Add: DBCPConnectionPool
   Name: MySQL USGS Database
   ```

3. **Configure Connection Pool**:
   ```properties
   Database Connection URL: jdbc:mysql://localhost:3306/usgs
   Database Driver Class Name: com.mysql.cj.jdbc.Driver
   Database Driver Location(s): /path/to/mysql-connector-java-8.0.x.jar
   Database User: nifi_user
   Password: your_password
   Max Wait Time: 500 millis
   Max Total Connections: 8
   Validation query: SELECT 1
   ```

**Important Notes**:

- **JDBC Driver**: Must be downloaded separately
  - Download from: https://dev.mysql.com/downloads/connector/j/
  - Extract JAR file
  - Place in accessible directory
  - Provide full path in "Database Driver Location(s)"

- **Connection String Format**:
  ```
  jdbc:mysql://[host]:[port]/[database]
  jdbc:mysql://localhost:3306/usgs
  jdbc:mysql://192.168.1.100:3306/usgs
  ```

- **Max Total Connections**: 
  - Should match or exceed concurrent tasks
  - Default: 8 (good for most cases)

4. Click **Apply**
5. Click **Enable**
6. Verify: No errors shown (green checkmark)

##### Create MySQL Table

Before proceeding, create the destination table:

```sql
CREATE DATABASE IF NOT EXISTS usgs;
USE usgs;

CREATE TABLE earthquakes (
    id VARCHAR(100),
    event_timestamp VARCHAR(100),
    latitude VARCHAR(50),
    longitude VARCHAR(50),
    depth VARCHAR(50),
    mag VARCHAR(50),
    magType VARCHAR(50),
    nst VARCHAR(50),
    gap VARCHAR(50),
    dmin VARCHAR(50),
    rms VARCHAR(50),
    net VARCHAR(50),
    updated VARCHAR(100),
    place VARCHAR(255),
    type VARCHAR(50),
    horizontalError VARCHAR(50),
    depthError VARCHAR(50),
    magError VARCHAR(50),
    magNst VARCHAR(50),
    status VARCHAR(50),
    locationSource VARCHAR(50),
    magSource VARCHAR(50),
    PRIMARY KEY (id)
);
```

**Note**: Using VARCHAR for simplicity. In production, use appropriate data types.

#### Configure ConvertJSONtoSQL

1. **Add Processor**: `ConvertJSONtoSQL`
2. **Rename**: `Convert JSON to SQL`
3. **Connect**: ConvertRecord (success) → ConvertJSONtoSQL

4. **Properties**:
   ```properties
   JDBC Connection Pool: MySQL USGS Database
   Statement Type: INSERT
   Table Name: earthquakes
   Catalog Name: usgs
   Schema Name: (leave empty for MySQL)
   Translate Field Names: true
   Unmatched Field Behavior: Ignore Unmatched Fields
   Unmatched Column Behavior: Fail on Unmatched Columns
   Update Keys: (leave empty - not doing updates)
   Field Containing SQL: (leave empty)
   SQL Parameter Attribute Prefix: sql
   Table Schema Cache Size: 100
   ```

**Key Properties Explained**:

- **Statement Type**: 
  - INSERT: Create new records
  - UPDATE: Modify existing records
  - UPSERT: Insert or update
  - DELETE: Remove records

- **Translate Field Names**: 
  - true: Attempt to match JSON field names to columns
  - false: Exact match required

- **Unmatched Field Behavior**:
  - Ignore: Skip fields not in table
  - Warn: Log warning but continue
  - Fail: Route to failure

5. **Settings Tab**:
   ```
   Automatically Terminate Relationships:
   ☑ failure
   ☑ original
   ```

#### How It Works

Input JSON:
```json
{"time":"2019-12-01T00:00:00.000Z","latitude":"36.1234"}
```

Output SQL Statement (in flow file content):
```sql
INSERT INTO earthquakes (time, latitude, ...) VALUES (?, ?, ...)
```

Output Attributes (parameter values):
```
sql.args.1.type = 12 (VARCHAR)
sql.args.1.value = 2019-12-01T00:00:00.000Z
sql.args.2.type = 12
sql.args.2.value = 36.1234
...
```

#### Testing Convert

1. Start ConvertJSONtoSQL
2. View output flow file
3. Check **Content**: Should see INSERT statement with ? placeholders
4. Check **Attributes**: Should see sql.args.X.value attributes

### Step 6: Put SQL (PutSQL Processor)

**Purpose**: Execute SQL statements against database

#### Configuration

1. **Add Processor**: `PutSQL`
2. **Rename**: `Put SQL to Earthquakes`
3. **Connect**: ConvertJSONtoSQL (sql) → PutSQL

4. **Properties**:
   ```properties
   JDBC Connection Pool: MySQL USGS Database
   SQL Statement: (leave empty - reads from flow file content)
   Batch Size: 1000
   Obtain Generated Keys: false
   Rollback On Failure: false
   Transaction Timeout: (leave empty)
   ```

**Batch Size Explained**:
- Groups multiple INSERT statements into one transaction
- Higher = Faster, but less granular error handling
- Lower = Slower, but better error isolation
- 100-1000 is typical

5. **Settings Tab**:
   ```
   Automatically Terminate Relationships:
   ☑ success
   ☑ failure
   ```

6. **Optional - Retry on Failure**:
   ```
   Instead of terminating failure:
   Connect: PutSQL (retry) → PutSQL (creates loop)
   ```

### Complete Flow Diagram

```
┌──────────────┐
│  Get USGS    │
│   Files      │
└──────┬───────┘
       │ success
       ▼
┌──────────────┐
│  Split Text  │
│    USGS      │
└──────┬───────┘
       │ splits
       ▼
┌──────────────┐
│Convert Record│
│ Text to JSON │
└──────┬───────┘
       │ success
       ▼
┌──────────────┐
│Convert JSON  │
│   to SQL     │
└──────┬───────┘
       │ sql
       ▼
┌──────────────┐
│   Put SQL    │
│to Earthquakes│
└──────────────┘
```

## Testing the Complete Flow

### Initial Test (Controlled)

1. **Setup**:
   ```
   GetFile: Batch Size = 1
   GetFile: Run Schedule = 60 sec
   GetFile: Keep Source File = true
   ```

2. **Execute**:
   - Start GetFile (wait for 1 file)
   - Stop GetFile
   - Start remaining processors
   - Watch flow files progress

3. **Verify**:
   ```sql
   SELECT COUNT(*) FROM usgs.earthquakes;
   SELECT * FROM usgs.earthquakes LIMIT 10;
   ```

### Production Run

1. **Adjust Settings**:
   ```
   GetFile:
   - Batch Size: 10
   - Run Schedule: 0 sec (continuous)
   - Keep Source File: false
   
   ConvertRecord:
   - Concurrent Tasks: 5
   
   PutSQL:
   - Batch Size: 1000
   - Concurrent Tasks: 1
   ```

2. **Monitor**:
   - Watch queue sizes
   - Check for backpressure
   - Monitor error rates
   - Verify database records

3. **Performance Tuning**:
   - Increase concurrent tasks if bottlenecked
   - Adjust batch sizes
   - Monitor system resources

## Troubleshooting

### Common Issues

#### 1. No Files Being Read

**Symptoms**: GetFile runs but no flow files created

**Solutions**:
- Check Input Directory path is correct
- Verify NiFi has read permissions on directory
- Confirm files match File Filter regex
- Check Minimum File Age isn't too restrictive

#### 2. CSV Parsing Errors

**Symptoms**: ConvertRecord fails with parsing errors

**Solutions**:
- Verify CSV header matches data
- Check for special characters in data
- Adjust Quote Character setting
- Enable Trim Fields option

#### 3. Database Connection Failures

**Symptoms**: "Cannot get connection" errors

**Solutions**:
- Verify database is running and accessible
- Check connection string format
- Confirm username/password
- Ensure JDBC driver path is correct
- Test connection pool (enable, check for errors)

#### 4. Column Mismatch Errors

**Symptoms**: "Column not found" in PutSQL

**Solutions**:
- Verify table exists
- Check column names match JSON fields
- Use Translate Field Names: true
- Adjust Unmatched Field Behavior

#### 5. Memory Issues

**Symptoms**: OutOfMemoryError, slow performance

**Solutions**:
- Reduce Batch Size in GetFile
- Increase NiFi heap size (nifi.properties)
- Add backpressure thresholds on connections
- Reduce concurrent tasks

### Checking Data Provenance

1. Right-click any queue
2. Select **Data provenance**
3. Click **Info** icon on event
4. View complete lineage:
   - Original filename
   - All transformations
   - Timing information
   - Attributes at each step

## Performance Optimization

### Concurrent Processing

```
GetFile: 1 thread (avoid duplicates if not clustered)
SplitText: 3-5 threads
ConvertRecord: 5-10 threads
ConvertJSONtoSQL: 5-10 threads
PutSQL: 1-3 threads (matches connection pool)
```

### Backpressure Configuration

Right-click connection → Configure:
```
Object Threshold: 10000
Size Threshold: 1 GB
```

### Batch Processing

```
GetFile Batch Size: 10-50 (how many files per run)
PutSQL Batch Size: 100-1000 (how many inserts per transaction)
```

## Best Practices

### 1. Error Handling

Always handle failure relationships:
- Log to file
- Route to separate error flow
- Send notifications
- Retry with exponential backoff

### 2. Monitoring

Add UpdateAttribute processors to:
- Track processing stages
- Add timestamps
- Include flow identifiers

### 3. Data Quality

Add validation processors:
- ValidateRecord (check schema)
- RouteOnAttribute (filter bad data)
- QueryRecord (data quality checks)

### 4. Documentation

- Use Labels to annotate complex logic
- Rename all processors descriptively
- Add comments in processor configurations
- Version control your flows (templates)

```
I'll continue creating all the remaining comprehensive guides.

## 03_jolt_transformation.md

```markdown
# Data Transformation with Jolt Transform JSON

## Overview

This guide covers using the Jolt Transform JSON processor to manipulate and transform JSON data structures in Apache NiFi. Jolt is a powerful JSON-to-JSON transformation library that allows you to reshape, filter, and modify JSON without writing code.

## What is Jolt?

**Jolt** = JSON Object Language for Transformations

**Key Capabilities**:
- Rename fields
- Restructure nested data
- Filter/remove unwanted fields
- Combine or split fields
- Apply conditional transformations
- Format data for specific destinations

## When to Use Jolt

### ✅ Good Use Cases

- Cleaning up API responses (remove unnecessary fields)
- Preparing data for specific destinations (Elasticsearch, databases)
- Restructuring deeply nested JSON
- Combining multiple fields into one
- Renaming fields to match target schema

### ❌ Not Ideal For

- Complex business logic
- Mathematical calculations (use QueryRecord instead)
- Multi-source joins (use QueryRecord)
- Conditional logic based on multiple fields

## Prerequisites

- Understanding of JSON structure
- Basic NiFi flow creation knowledge
- Sample JSON data to transform

## Tutorial Scenario

**Goal**: Transform USGS earthquake data to prepare for Elasticsearch

### Input Data (from previous flow)

```json
{
  "time": "2020-02-07 15:14:57",
  "latitude": "36.1234",
  "longitude": "-117.5678",
  "depth": "2.3",
  "mag": "1.2",
  "magType": "ml",
  "nst": "12",
  "gap": "45",
  "dmin": "0.01234",
  "rms": "0.12",
  "net": "ci",
  "id": "ci12345678",
  "updated": "2020-02-07 15:19:57",
  "place": "9km SE of Ridgecrest, CA",
  "type": "earthquake",
  "horizontalError": "0.5",
  "depthError": "1.2",
  "magError": "0.1",
  "magNst": "5",
  "status": "automatic",
  "locationSource": "ci",
  "magSource": "ci"
}
```

### Desired Output

```json
{
  "event_timestamp": "2020-02-07 15:14:57",
  "geolocation": "36.1234,-117.5678",
  "depth": "2.3",
  "magnitude": "1.2",
  "id": "ci12345678",
  "place": "9km SE of Ridgecrest, CA",
  "type": "earthquake"
}
```

### Transformation Goals

1. ✅ Keep only needed fields (7 out of 21)
2. ✅ Rename `time` → `event_timestamp`
3. ✅ Rename `mag` → `magnitude`
4. ✅ Combine `latitude` + `longitude` → `geolocation`
5. ✅ Remove all other fields

## Flow Setup

### Starting Point

From the previous CSV to MySQL flow:

```
GetFile → SplitText → ConvertRecord → [INSERT JOLT HERE] → ConvertJSONtoSQL → PutSQL
```

### Adding Jolt Processor

1. **Stop the flow**
2. **Delete connection**: ConvertRecord → ConvertJSONtoSQL
3. **Add processor**: Search "Jolt Transform JSON"
4. **Connect**:
   ```
   ConvertRecord (success) → JoltTransformJSON
   JoltTransformJSON (success) → ConvertJSONtoSQL
   ```

## Jolt Configuration

### Processor Settings

1. **Rename**: `Jolt Transform JSON` → `Transform JSON - Clean USGS Data`

2. **Scheduling**:
   ```
   Run Schedule: 0 sec (immediate processing)
   Concurrent Tasks: 5
   ```

3. **Properties**:
   ```properties
   Jolt Transformation DSL: jolt-transform-chain
   Jolt Specification: (see below)
   Custom Transformation Class Name: (leave empty)
   Custom Module Directory: (leave empty)
   Transform Cache Size: 1
   ```

### Jolt Transformation DSL Options

- **jolt-transform-chain**: Sequential transformations (most common)
- **jolt-transform-shift**: Move/copy data
- **jolt-transform-default**: Add default values
- **jolt-transform-remove**: Remove fields
- **jolt-transform-sort**: Sort keys
- **jolt-transform-modify-default**: Modify with defaults
- **jolt-transform-modify-overwrite**: Overwrite values
- **jolt-transform-modify-define**: Define new values
- **jolt-transform-card**: Cardinality operations

We'll use **chain** to combine multiple operations.

## Writing Jolt Specifications

### Method 1: Built-in Editor

1. Click **Advanced** button in Jolt Specification property
2. Opens a test editor with:
   - **Input** tab: Paste sample JSON
   - **Spec** tab: Write Jolt specification
   - **Transform** button: Test transformation
   - **Output** tab: View results

### Method 2: External Testing

Use online tool: https://jolt-demo.appspot.com

This provides:
- Syntax highlighting
- Real-time validation
- Example transformations
- Better error messages

## Jolt Specification Breakdown

### Complete Specification

```json
[
  {
    "operation": "shift",
    "spec": {
      "time": "event_timestamp",
      "latitude": "geolocation[0]",
      "longitude": "geolocation[1]",
      "depth": "depth",
      "mag": "magnitude",
      "id": "id",
      "place": "place",
      "type": "type"
    }
  },
  {
    "operation": "modify-overwrite-beta",
    "spec": {
      "geolocation": "=toString"
    }
  },
  {
    "operation": "modify-overwrite-beta",
    "spec": {
      "geolocation": "=join(',',@(1,geolocation))"
    }
  }
]
```

### Step-by-Step Explanation

#### Operation 1: Shift (Rename and Restructure)

```json
{
  "operation": "shift",
  "spec": {
    "time": "event_timestamp",
    "latitude": "geolocation[0]",
    "longitude": "geolocation[1]",
    "depth": "depth",
    "mag": "magnitude",
    "id": "id",
    "place": "place",
    "type": "type"
  }
}
```

**What it does**:
- **Rename**: `time` becomes `event_timestamp`
- **Rename**: `mag` becomes `magnitude`
- **Keep**: `depth`, `id`, `place`, `type` unchanged
- **Combine**: Create `geolocation` array with lat/long
- **Remove**: Everything not listed is discarded

**After Operation 1**:
```json
{
  "event_timestamp": "2020-02-07 15:14:57",
  "geolocation": ["36.1234", "-117.5678"],
  "depth": "2.3",
  "magnitude": "1.2",
  "id": "ci12345678",
  "place": "9km SE of Ridgecrest, CA",
  "type": "earthquake"
}
```

#### Operation 2: Convert Array to String

```json
{
  "operation": "modify-overwrite-beta",
  "spec": {
    "geolocation": "=toString"
  }
}
```

**What it does**:
- Converts array to string representation

**After Operation 2**:
```json
{
  "geolocation": "[36.1234, -117.5678]"
  ...
}
```

**Note**: This isn't quite what we want yet (has brackets and space).

#### Operation 3: Join with Comma

```json
{
  "operation": "modify-overwrite-beta",
  "spec": {
    "geolocation": "=join(',',@(1,geolocation))"
  }
}
```

**What it does**:
- Uses `join()` function to combine array elements with comma
- `@(1,geolocation)`: Reference the geolocation field

**After Operation 3** (Final):
```json
{
  "event_timestamp": "2020-02-07 15:14:57",
  "geolocation": "36.1234,-117.5678",
  "depth": "2.3",
  "magnitude": "1.2",
  "id": "ci12345678",
  "place": "9km SE of Ridgecrest, CA",
  "type": "earthquake"
}
```

✅ Perfect! This is our desired output.

## Testing Jolt Transformations

### Using Built-in Editor

1. **Open Jolt Specification property**
2. **Click Advanced button**
3. **Input Tab**: Paste sample JSON
   ```json
   {
     "time": "2020-02-07 15:14:57",
     "latitude": "36.1234",
     "longitude": "-117.5678",
     "mag": "1.2",
     ...
   }
   ```
4. **Spec Tab**: Write transformation
5. **Click Transform**
6. **Output Tab**: Verify results
7. **If correct**: Click Save

### Using External Tool

1. Go to: https://jolt-demo.appspot.com
2. **Input** (left): Paste JSON
3. **Spec** (middle): Paste specification
4. **Transform** button
5. **Output** (right): View results
6. Iterate until correct
7. Copy spec to NiFi

## Common Jolt Patterns

### 1. Simple Rename

**Input**:
```json
{"oldName": "value"}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "oldName": "newName"
  }
}
```

**Output**:
```json
{"newName": "value"}
```

### 2. Keep Only Specific Fields

**Input**:
```json
{
  "field1": "keep",
  "field2": "keep",
  "field3": "remove",
  "field4": "remove"
}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "field1": "field1",
    "field2": "field2"
  }
}
```

**Output**:
```json
{
  "field1": "keep",
  "field2": "keep"
}
```

### 3. Flatten Nested Objects

**Input**:
```json
{
  "user": {
    "name": "John",
    "email": "john@example.com"
  }
}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "user": {
      "name": "userName",
      "email": "userEmail"
    }
  }
}
```

**Output**:
```json
{
  "userName": "John",
  "userEmail": "john@example.com"
}
```

### 4. Create Nested Structure

**Input**:
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "city": "New York"
}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "firstName": "user.name.first",
    "lastName": "user.name.last",
    "city": "user.address.city"
  }
}
```

**Output**:
```json
{
  "user": {
    "name": {
      "first": "John",
      "last": "Doe"
    },
    "address": {
      "city": "New York"
    }
  }
}
```

### 5. Convert Array Elements

**Input**:
```json
{
  "items": [
    {"name": "A", "price": 10},
    {"name": "B", "price": 20}
  ]
}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "items": {
      "*": {
        "name": "products[&1].productName",
        "price": "products[&1].cost"
      }
    }
  }
}
```

**Output**:
```json
{
  "products": [
    {"productName": "A", "cost": 10},
    {"productName": "B", "cost": 20}
  ]
}
```

**Wildcards Explained**:
- `*`: Matches any element
- `&1`: References the matched index
- `&(1,0)`: Go up 1 level, grab index 0

### 6. Add Default Values

**Input**:
```json
{
  "name": "John"
}
```

**Spec**:
```json
[
  {
    "operation": "shift",
    "spec": {
      "*": "&"
    }
  },
  {
    "operation": "default",
    "spec": {
      "status": "active",
      "role": "user"
    }
  }
]
```

**Output**:
```json
{
  "name": "John",
  "status": "active",
  "role": "user"
}
```

### 7. Conditional Transformations

**Input**:
```json
{
  "type": "premium",
  "price": 100
}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "type": {
      "premium": {
        "@(2,price)": "premiumPrice"
      },
      "*": {
        "@(2,price)": "regularPrice"
      }
    }
  }
}
```

**Output** (when type=premium):
```json
{"premiumPrice": 100}
```

### 8. String Manipulation

**Input**:
```json
{
  "fullName": "John Doe"
}
```

**Spec**:
```json
{
  "operation": "modify-overwrite-beta",
  "spec": {
    "fullName": "=toUpper"
  }
}
```

**Output**:
```json
{"fullName": "JOHN DOE"}
```

**Available Functions**:
- `=toUpper`: Convert to uppercase
- `=toLower`: Convert to lowercase
- `=toString`: Convert to string
- `=toInteger`: Convert to integer
- `=toDouble`: Convert to double
- `=toBoolean`: Convert to boolean
- `=size`: Get array/string size
- `=join(',',@(1,fieldName))`: Join array with delimiter

## Advanced Jolt Techniques

### Multiple Operations (Chain)

Combine multiple transformation types:

```json
[
  {
    "operation": "shift",
    "spec": {
      "user": {
        "*": "&"
      }
    }
  },
  {
    "operation": "default",
    "spec": {
      "status": "active"
    }
  },
  {
    "operation": "modify-overwrite-beta",
    "spec": {
      "name": "=toUpper"
    }
  }
]
```

### Working with Arrays

**Input**:
```json
{
  "users": [
    {"id": 1, "name": "John"},
    {"id": 2, "name": "Jane"}
  ]
}
```

**Spec** (Extract names only):
```json
{
  "operation": "shift",
  "spec": {
    "users": {
      "*": {
        "name": "names[]"
      }
    }
  }
}
```

**Output**:
```json
{
  "names": ["John", "Jane"]
}
```

### Reference Other Fields

Use `@` notation to reference values:

```json
{
  "operation": "shift",
  "spec": {
    "firstName": "user.name.first",
    "firstName": "user.display",  // Use same value twice
    "age": {
      "*": {
        "@(1,firstName)": "ageGroups.&.name"
      }
    }
  }
}
```

**@ Notation**:
- `@(1,fieldName)`: Go up 1 level, grab fieldName
- `@(2,fieldName)`: Go up 2 levels, grab fieldName
- `@`: Current value

### Remove Specific Fields

```json
{
  "operation": "remove",
  "spec": {
    "unwantedField": "",
    "anotherBadField": ""
  }
}
```

### Sort Keys

```json
{
  "operation": "sort"
}
```

## Integration in NiFi Flow

### Complete Flow with Jolt

```
GetFile
  ↓
SplitText
  ↓
ConvertRecord (CSV → JSON)
  ↓
JoltTransformJSON (Clean & Reshape)
  ↓
ConvertJSONtoSQL (or other destination)
  ↓
PutSQL
```

### Testing in Flow

1. **Stop all processors**
2. **Start upstream** (GetFile, SplitText, ConvertRecord)
3. **Let queue fill** before Jolt
4. **List queue**:
   - View input JSON
   - Verify structure
5. **Start Jolt processor**
6. **List output queue**:
   - View transformed JSON
   - Verify correctness
7. **Iterate** if needed:
   - Stop Jolt
   - Modify specification
   - Empty output queue
   - Restart Jolt
   - Verify again

### Error Handling

**Settings Tab**:
```
Automatically Terminate Relationships:
☐ success (connect to next processor)
☑ failure (or connect to error handling)
```

**Failure Handling Options**:
1. **Terminate**: Discard failed transformations
2. **Log**: Route to LogAttribute for debugging
3. **Retry**: Loop back with modified spec
4. **Alert**: Send to notification processor

## Best Practices

### 1. Start Simple

Begin with basic shift operation:
```json
{
  "operation": "shift",
  "spec": {
    "*": "&"  // Pass through everything
  }
}
```

Then add complexity incrementally.

### 2. Test Incrementally

- Add one operation at a time
- Test after each addition
- Verify output at each step

### 3. Use Comments (Not in Spec)

Document your transformations outside NiFi:

```javascript
// Operation 1: Rename fields and keep only needed data
// Operation 2: Convert geolocation array to string
// Operation 3: Format geolocation as comma-separated
```

### 4. Handle Nulls

Consider null values in your spec:

```json
{
  "operation": "default",
  "spec": {
    "fieldName": "defaultValue"
  }
}
```

### 5. Version Control

- Save specs to external files
- Use git for version tracking
- Document transformation purpose

### 6. Performance Considerations

- Jolt is fast but complex specs can slow processing
- For very complex logic, consider QueryRecord
- Monitor processor performance metrics

### 7. Validate Output

Always verify:
- Field names match expectations
- Data types are correct
- No data loss occurred
- Format matches destination requirements

## Troubleshooting

### Common Errors

#### 1. "Invalid Jolt Specification"

**Cause**: JSON syntax error in spec

**Solution**:
- Validate JSON syntax
- Check for missing commas, brackets
- Use JSON validator tool

#### 2. "Transformation produces no output"

**Cause**: Spec doesn't match input structure

**Solution**:
- Verify input JSON structure
- Check field name spelling
- Test with sample in editor

#### 3. "Output has unexpected structure"

**Cause**: Wildcards or references incorrect

**Solution**:
- Review wildcard usage
- Check `@` reference levels
- Test step-by-step

#### 4. "Array not combining correctly"

**Cause**: Wrong join syntax or operation order

**Solution**:
- Ensure array exists before joining
- Use correct function syntax
- Check operation sequence

### Debugging Techniques

1. **Use Built-in Editor**:
   - Paste actual flow file content
   - Test transformation
   - View detailed output

2. **Add LogAttribute**:
   ```
   JoltTransformJSON → LogAttribute
   ```
   Set to log payload to see results

3. **Simplify Spec**:
   - Remove operations one by one
   - Identify problematic section
   - Fix and re-add

4. **Check Provenance**:
   - View before/after content
   - Compare attributes
   - Track transformation history

## Real-World Examples

### Example 1: API Response Cleanup

**Input** (Twitter API):
```json
{
  "created_at": "Mon Feb 03 15:30:00 +0000 2020",
  "id_str": "1224362147",
  "text": "Sample tweet",
  "user": {
    "id": 12345,
    "name": "John Doe",
    "followers_count": 1000
  },
  "retweet_count": 5,
  "favorite_count": 10
}
```

**Spec**:
```json
{
  "operation": "shift",
  "spec": {
    "created_at": "timestamp",
    "id_str": "tweet_id",
    "text": "message",
    "user": {
      "name": "author",
      "followers_count": "author_followers"
    },
    "retweet_count": "retweets",
    "favorite_count": "likes"
  }
}
```

**Output**:
```json
{
  "timestamp": "Mon Feb 03 15:30:00 +0000 2020",
  "tweet_id": "1224362147",
  "message": "Sample tweet",
  "author": "John Doe",
  "author_followers": 1000,
  "retweets": 5,
  "likes": 10
}
```

### Example 2: Elasticsearch Geo-Point Format

**Input**:
```json
{
  "location_lat": 40.7128,
  "location_lon": -74.0060,
  "city": "New York"
}
```

**Spec**:
```json
[
  {
    "operation": "shift",
    "spec": {
      "location_lat": "location.lat",
      "location_lon": "location.lon",
      "city": "city"
    }
  }
]
```

**Output** (Elasticsearch compatible):
```json
{
  "location": {
    "lat": 40.7128,
    "lon": -74.0060
  },
  "city": "New York"
}
```

### Example 3: Database Denormalization

**Input** (Normalized):
```json
{
  "order_id": 12345,
  "customer": {
    "id": 67890,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "items": [
    {"product": "Widget", "qty": 2},
    {"product": "Gadget", "qty": 1}
  ]
}
```

**Spec** (Flatten for analytics):
```json
{
  "operation": "shift",
  "spec": {
    "order_id": "order_id",
    "customer": {
      "id": "customer_id",
      "name": "customer_name",
      "email": "customer_email"
    },
    "items": {
      "*": {
        "product": "items[&1].product",
        "qty": "items[&1].quantity"
      }
    }
  }
}
```

## Comparison: Jolt vs QueryRecord

| Feature | Jolt | QueryRecord |
|---------|------|-------------|
| **Best For** | Structural changes | Calculations, filtering |
| **Syntax** | JSON specification | SQL queries |
| **Learning Curve** | Moderate | Easy (if SQL knowledge) |
| **Performance** | Fast | Fast |
| **Complexity Limit** | Medium | High |
| **Field Operations** | Rename, restructure | Calculate, aggregate |
| **Multi-field Logic** | Limited | Excellent |
| **Debugging** | Moderate | Easy |

**When to Choose Jolt**:
- Renaming many fields
- Restructuring nested JSON
- Removing unwanted fields
- Format conversion

**When to Choose QueryRecord**:
- Mathematical operations
- Conditional logic
- Aggregations
- Filtering based on values

## Additional Resources

### Online Tools

- **Jolt Demo**: https://jolt-demo.appspot.com
- **JSON Formatter**: https://jsonformatter.org
- **JSON Validator**: https://jsonlint.com

### Documentation

- **Jolt GitHub**: https://github.com/bazaarvoice/jolt
- **NiFi Jolt Processor**: Access via "View Usage" in NiFi
- **Expression Language**: NiFi docs for modify operations

### Example Specifications

Many examples available at:
- NiFi template exchange
- GitHub repositories
- Community forums

## Summary

### Key Takeaways

✅ **Jolt** is powerful for JSON restructuring
✅ **Chain** multiple operations for complex transformations
✅ **Test** specifications before deploying
✅ **Use built-in editor** or online tools
✅ **Start simple** and add complexity gradually
✅ **Document** your transformations
✅ **Consider alternatives** (QueryRecord) for complex logic

### Skills Learned

- Writing Jolt specifications
- Using shift operations
- Combining fields
- Renaming and filtering
- Testing transformations
- Integrating in NiFi flows

### Next Steps

- Practice with your own data
- Explore [QueryRecord](06_query_record.md)
- Learn [Advanced Data Processing](advanced_patterns.md)
- Try [ConvertRecord alternatives](convert_record_guide.md)
```

## 04_creating_csv_files.md

```markdown
# Creating CSV Files from NiFi Flows

## Overview

This guide demonstrates how to take JSON data from a flow and convert it back to CSV format, then write it to the filesystem. This is useful for:
- Exporting processed data
- Creating reports
- Feeding data to third-party systems
- Archiving transformed data

## Tutorial Scenario

**Goal**: Extend the USGS earthquake flow to output cleaned CSV files

**Starting Point**: Flow with Jolt-transformed JSON data

**End Result**: CSV files written to filesystem with cleaned data

## Flow Architecture

```
[Previous Flow: Get → Split → Convert → Jolt]
          ↓
    EvaluateJsonPath (Extract attributes)
          ↓
    AttributesToCSV (Convert to CSV format)
          ↓
    MergeContent (Combine records)
          ↓
    UpdateAttribute (Set filename)
          ↓
    PutFile (Write to disk)
```

## Prerequisites

- Completed Jolt transformation tutorial
- JSON data in flow
- Write access to output directory

## Step-by-Step Implementation

### Step 1: Evaluate JSON Path

**Purpose**: Extract JSON fields as flow file attributes

#### Why This Step?

The `AttributesToCSV` processor needs data in attributes, not content. We must extract JSON fields to attributes first.

#### Configuration

1. **Add Processor**: Search "EvaluateJsonPath"
2. **Rename**: `Extract JSON to Attributes`
3. **Connect**: Jolt (success) → EvaluateJsonPath

4. **Properties**:
   ```properties
   Destination: flowfile-attribute
   Return Type: auto-detect
   Path Not Found Behavior: ignore
   Null Value Representation: empty string
   ```

5. **Add Dynamic Properties** (for each field to extract):

   ```properties
   # Property Name → JsonPath Expression
   
   event_timestamp → $.event_timestamp
   geolocation → $.geolocation
   depth → $.depth
   magnitude → $.magnitude
   id → $.id
   place → $.place
   type → $.type
   ```

#### How to Add Dynamic Properties

1. Click **+** button in Properties tab
2. **Property Name**: The attribute name you want (e.g., "event_timestamp")
3. **Value**: JsonPath expression (e.g., "$.event_timestamp")
4. Click **OK**
5. Repeat for each field

#### JsonPath Syntax Quick Reference

```
$.fieldName          - Top-level field
$.nested.field       - Nested field
$.array[0]          - Array element
$.array[*]          - All array elements
$..fieldName        - Recursive search
```

#### Settings

```
Automatically Terminate Relationships:
☑ failure
☑ unmatched
```

6. **Apply** and close

#### Testing

1. Start EvaluateJsonPath
2. View output flow file
3. Check **Attributes** tab:
   ```
   event_timestamp: 2020-02-07 15:14:57
   geolocation: 36.1234,-117.5678
   depth: 2.3
   magnitude: 1.2
   id: ci12345678
   place: 9km SE of Ridgecrest, CA
   type: earthquake
   ```

✅ Success! JSON fields are now attributes.

### Step 2: Attributes to CSV

**Purpose**: Convert flow file attributes to CSV content

#### Configuration

1. **Add Processor**: Search "AttributesToCSV"
2. **Rename**: `Convert Attributes to CSV`
3. **Connect**: EvaluateJsonPath (matched) → AttributesToCSV

4. **Properties**:
   ```properties
   Attribute List: event_timestamp,geolocation,depth,magnitude,id,place,type
   Destination: flowfile-content
   Include Core Attributes: false
   Null Value: 
   Include Schema: false
   ```

**Property Details**:

- **Attribute List**: Comma-separated list (ORDER MATTERS!)
  ```
  event_timestamp,geolocation,depth,magnitude,id,place,type
  ```
  The order here determines column order in CSV

- **Destination**: 
  - `flowfile-content`: Replace content with CSV (what we want)
  - `flowfile-attribute`: Create new attribute with CSV

- **Include Core Attributes**: 
  - `false`: Only our custom attributes
  - `true`: Includes uuid, filename, etc.

- **Null Value**: What to write for null values (empty string)

- **Include Schema**: 
  - `false`: No header row added here
  - `true`: Adds header row

5. **Settings**:
   ```
   Automatically Terminate Relationships:
   ☑ failure
   ```

6. **Apply**

#### Testing

1. Start AttributesToCSV
2. View output content:
   ```csv
   2020-02-07 15:14:57,36.1234,-117.5678,2.3,1.2,ci12345678,9km SE of Ridgecrest, CA,earthquake
   ```

✅ Perfect! One CSV row per flow file.

### Step 3: Merge Content

**Purpose**: Combine multiple flow files into larger files

#### Why Merge?

- We have thousands of individual flow files
- Don't want thousands of tiny CSV files
- Better to have fewer, larger files
- Improves I/O efficiency

#### Configuration

1. **Add Processor**: Search "MergeContent"
2. **Rename**: `Merge CSV Records`
3. **Connect**: AttributesToCSV (success) → MergeContent

4. **Properties**:
   ```properties
   Merge Strategy: Bin-Packing Algorithm
   Merge Format: Binary Concatenation
   Correlation Attribute Name: (leave empty)
   Attribute Strategy: Keep Only Common Attributes
   Minimum Number of Entries: 1000
   Maximum Number of Entries: 500000
   Minimum Group Size: 0 B
   Maximum Group Size: (leave empty)
   Max Bin Age: 10 min
   Maximum Number of Bins: 100
   Delimiter Strategy: Text
   Header: event_timestamp,geolocation,depth,magnitude,id,place,type
   Footer: (leave empty)
   Demarcator: (Press Shift+Enter for newline)
   Compression Level: 0
   Keep Path: false
   ```

**Critical Properties Explained**:

##### Merge Strategy

- **Bin-Packing Algorithm**: Groups flow files until bins are full
- **Defragment**: Reassembles fragmented flow files (not needed here)

##### Merge Format

- **Binary Concatenation**: Simple concatenation (for text/CSV)
- **TAR**: Create TAR archive
- **ZIP**: Create ZIP archive
- **FlowFile Stream v3/v2/v1**: NiFi-specific formats

##### Minimum/Maximum Number of Entries

```properties
Minimum Number of Entries: 1000   # Wait for at least 1000 records
Maximum Number of Entries: 500000 # Don't exceed 500k records per file
```

This creates bins of 1,000-500,000 records.

##### Demarcator

**VERY IMPORTANT**: Must add newline character

**How to Add Newline**:
1. Click in Demarcator field
2. Press **Shift + Enter** (creates visible newline)
3. You should see the field has two lines now

Without this, all CSV rows will be on one line!

##### Header

```properties
Header: event_timestamp,geolocation,depth,magnitude,id,place,type
```

Adds CSV header row to beginning of each merged file.

##### Max Bin Age

```properties
Max Bin Age: 10 min
```

If bin doesn't fill in 10 minutes, flush it anyway.

5. **Settings**:
   ```
   Automatically Terminate Relationships:
   ☑ failure
   ☑ original
   ```

6. **Apply**

#### Understanding Merge Behavior

**Scenario**: 5,000 flow files arrive

**With Our Settings**:
- Bin 1: Fills to 1,000 records → Flushes
- Bin 2: Fills to 1,000 records → Flushes
- Bin 3: Fills to 1,000 records → Flushes
- Bin 4: Fills to 1,000 records → Flushes
- Bin 5: Has 1,000 records → Flushes
- Result: 5 merged files, 1,000 records each

**If only 500 arrive**:
- Bin 1: Has 500 records
- Waits 10 minutes (Max Bin Age)
- Flushes with 500 records

#### Testing

1. **Setup**: Ensure at least 1,000 flow files in queue
2. **Start** MergeContent
3. **Wait**: For minimum threshold
4. **View** merged flow file content:
   ```csv
   event_timestamp,geolocation,depth,magnitude,id,place,type
   2020-02-07 15:14:57,36.1234,-117.5678,2.3,1.2,ci12345678,9km SE of Ridgecrest CA,earthquake
   2020-02-07 15:15:02,36.1235,-117.5679,2.4,1.3,ci12345679,10km SE of Ridgecrest CA,earthquake
   ...
   [1000 total rows]
   ```

✅ Multiple records merged with header!

**Check Attributes**:
```
fragment.count: 1000      # How many were merged
merge.count: 1000         # Same information
merge.bin.age: 00:00:05   # How long bin was open
```

### Step 4: Update Attribute (Set Filename)

**Purpose**: Create meaningful filename with timestamp

#### Why?

Default filename is UUID (not helpful):
```
abc123-def456-ghi789
```

Better filename:
```
earthquakes_merged_20200207_151500.csv
```

#### Configuration

1. **Add Processor**: Search "UpdateAttribute"
2. **Rename**: `Set Output Filename`
3. **Connect**: MergeContent (merged) → UpdateAttribute

4. **Properties** (Add Dynamic Properties):

   ```properties
   # Property: filename
   # Value:
   earthquakes_merged_${now():format('yyyyMMdd_HHmmss')}.csv
   ```

**Expression Language Breakdown**:

```
earthquakes_merged_                    # Static text
${now():format('yyyyMMdd_HHmmss')}    # Dynamic timestamp
.csv                                   # File extension
```

**Functions Used**:
- `now()`: Current timestamp
- `format('yyyyMMdd_HHmmss')`: Format as YYYYMMdd_HHmmss

**Result**: `earthquakes_merged_20200207_151457.csv`

**Alternative Formats**:
```
${now():format('yyyy-MM-dd')}              → 2020-02-07
${now():format('yyyy/MM/dd HH:mm:ss')}     → 2020/02/07 15:14:57
${now():format('yyyyMMdd')}                → 20200207
${now():format('E MMM dd yyyy')}           → Fri Feb 07 2020
```

5. **Apply**

#### Testing

1. Start UpdateAttribute
2. View output attributes:
   ```
   filename: earthquakes_merged_20200207_151457.csv
   ```

### Step 5: Put File

**Purpose**: Write CSV files to filesystem

#### Configuration

1. **Add Processor**: Search "PutFile"
2. **Rename**: `Write CSV to Filesystem`
3. **Connect**: UpdateAttribute (success) → PutFile

4. **Properties**:
   ```properties
   Directory: /output/path/earthquakes
   Conflict Resolution Strategy: replace
   Create Missing Directories: true
   Maximum File Count: -1
   Last Modified Time: (leave empty)
   Permissions: (leave empty)
   Owner: (leave empty)
   Group: (leave empty)
   Directory Permissions: (leave empty)
   ```

**Property Details**:

##### Directory

Absolute path where files will be written:
```
Linux/Mac: /data/output/earthquakes
Windows: C:/data/output/earthquakes
Docker: /mnt/output/earthquakes (must be mounted)
```

##### Conflict Resolution Strategy

- **replace**: Overwrite if file exists
- **ignore**: Skip if file exists
- **fail**: Route to failure if exists

##### Create Missing Directories

- `true`: Create directory structure if doesn't exist
- `false`: Fail if directory doesn't exist

5. **Settings**:
   ```
   Automatically Terminate Relationships:
   ☑ success
   ☑ failure
   ```

6. **Apply**

#### Testing

1. **Verify Directory Access**:
   ```bash
   # Ensure NiFi has write permissions
   ls -la /output/path
   chmod 755 /output/path/earthquakes
   ```

2. **Start PutFile**
3. **Check filesystem**:
   ```bash
   ls -lh /output/path/earthquakes/
   # Should see:
   # earthquakes_merged_20200207_151457.csv
   ```

4. **Verify content**:
   ```bash
   head -5 /output/path/earthquakes/earthquakes_merged_20200207_151457.csv
   ```

Expected output:
```csv
event_timestamp,geolocation,depth,magnitude,id,place,type
2020-02-07 15:14:57,36.1234,-117.5678,2.3,1.2,ci12345678,9km SE of Ridgecrest CA,earthquake
2020-02-07 15:15:02,36.1235,-117.5679,2.4,1.3,ci12345679,10km SE of Ridgecrest CA,earthquake
...
```

✅ CSV files successfully created!

## Complete Flow Diagram

```
┌─────────────┐
│   GetFile   │
│  (Source)   │
└──────┬──────┘
       │
┌──────▼──────────┐
│   SplitText     │
│ (One row each)  │
└──────┬──────────┘
       │
┌──────▼──────────┐
│ ConvertRecord   │
│  (CSV → JSON)   │
└──────┬──────────┘
       │
┌──────▼──────────┐
│ JoltTransform   │
│ (Clean data)    │
└──────┬──────────┘
       │
       ├───────────────────────┐
       │                       │
┌──────▼──────────┐   ┌────────▼─────────┐
│ConvertJSONtoSQL │   │EvaluateJsonPath  │
│                 │   │                  │
└──────┬──────────┘   └────────┬─────────┘
       │                       │
┌──────▼──────────┐   ┌────────▼─────────┐
│    PutSQL       │   │AttributesToCSV   │
│  (To MySQL)     │   │                  │
└─────────────────┘   └────────┬─────────┘
                               │
                      ┌────────▼─────────┐
                      │  MergeContent    │
                      │ (Combine 1000)   │
                      └────────┬─────────┘
                               │
                      ┌────────▼─────────┐
                      │UpdateAttribute   │
                      │ (Set filename)   │
                      └────────┬─────────┘
                               │
                      ┌────────▼─────────┐
                      │    PutFile       │
                      │ (Write to disk)  │
                      └──────────────────┘
```

## Advanced Patterns

### Pattern 1: Dynamic Directory Structure

Create date-based directory structure:

**UpdateAttribute Configuration**:
```properties
# Create directory attribute
directory: /output/${now():format('yyyy/MM/dd')}

# Or use source date
directory: /output/${event_timestamp:toDate('yyyy-MM-dd HH:mm:ss'):format('yyyy/MM/dd')}
```

**PutFile Configuration**:
```properties
Directory: ${directory}
Create Missing Directories: true
```

**Result**:
```
/output/2020/02/07/earthquakes_merged_151457.csv
/output/2020/02/08/earthquakes_merged_093022.csv
```

### Pattern 2: Conditional File Routing

Route to different directories based on criteria:

**RouteOnAttribute** (before PutFile):
```properties
# Property: high_magnitude
# Value: ${magnitude:toNumber():gt(5.0)}

# Property: medium_magnitude  
# Value: ${magnitude:toNumber():ge(3.0):and(${magnitude:toNumber():lt(5.0)})}

# Property: low_magnitude
# Value: ${magnitude:toNumber():lt(3.0)}
```

**Multiple PutFile Processors**:
```
RouteOnAttribute → PutFile (high) → /output/high-magnitude/
                ├→ PutFile (medium) → /output/medium-magnitude/
                └→ PutFile (low) → /output/low-magnitude/
```

### Pattern 3: Compressed Output

**MergeContent Configuration**:
```properties
Merge Format: ZIP
Compression Level: 9
```

**UpdateAttribute**:
```properties
filename: earthquakes_merged_${now():format('yyyyMMdd_HHmmss')}.zip
```

### Pattern 4: Split by Size

Create multiple files if data exceeds size limit:

**MergeContent Configuration**:
```properties
Minimum Number of Entries: 1000
Maximum Number of Entries: 10000
Maximum Group Size: 10 MB
```

This creates multiple merged files if either:
- 10,000 records reached, OR
- 10 MB size reached

### Pattern 5: Add Custom Footer

**MergeContent Configuration**:
```properties
Footer: Total Records: ${fragment.count}
```

**Result**:
```csv
event_timestamp,geolocation,depth,magnitude,id,place,type
2020-02-07 15:14:57,36.1234,-117.5678,2.3,1.2,ci12345678,...
...
Total Records: 1000
```

## Optimization Tips

### Performance Tuning

**For High Volume**:
```properties
EvaluateJsonPath:
  Concurrent Tasks: 5

AttributesToCSV:
  Concurrent Tasks: 5

MergeContent:
  Concurrent Tasks: 1  # Keep at 1 to avoid bin conflicts
  Minimum Number of Entries: 5000
  Maximum Number of Bins: 10

PutFile:
  Concurrent Tasks: 3
```

### Memory Management

**Large Files**:
- Increase NiFi heap size in `nifi.properties`
- Reduce Maximum Number of Entries
- Enable backpressure on connections

**Backpressure Example**:
```
Right-click connection → Configure
Object Threshold: 10000
Size Threshold: 100 MB
```

### Disk I/O

**Reduce Disk Writes**:
- Increase merge batch sizes
- Use faster storage for content repository
- Enable provenance repository on SSD

## Troubleshooting

### Issue 1: No Newlines in CSV

**Symptom**: All records on one line

**Cause**: Missing demarcator in MergeContent

**Solution**:
1. Stop MergeContent
2. Edit properties
3. Click in Demarcator field
4. Press **Shift + Enter**
5. Apply and restart

### Issue 2: Missing Header

**Symptom**: No header row in CSV

**Cause**: Header property empty in MergeContent

**Solution**:
```properties
Header: event_timestamp,geolocation,depth,magnitude,id,place,type
```

### Issue 3: Files Not Writing

**Symptom**: PutFile shows success but no files appear

**Causes & Solutions**:

1. **Permission Issue**:
   ```bash
   # Check directory permissions
   ls -la /output/path
   
   # Fix permissions
   chmod 755 /output/path
   chown nifi:nifi /output/path
   ```

2. **Path Doesn't Exist**:
   ```properties
   Create Missing Directories: true
   ```

3. **Wrong Path**:
   - Verify absolute path
   - Check for typos
   - Ensure mount point exists (Docker)

### Issue 4: Attribute Not Found

**Symptom**: EvaluateJsonPath failure

**Cause**: JsonPath doesn't match JSON structure

**Solution**:
1. List queue before EvaluateJsonPath
2. View flow file content
3. Verify JSON structure
4. Update JsonPath expressions

**Example**:
```
If JSON is:
{"data": {"event_timestamp": "..."}}

JsonPath should be:
$.data.event_timestamp
```

### Issue 5: CSV Column Order Wrong

**Symptom**: Columns in wrong order

**Cause**: Attribute List order doesn't match Header

**Solution**:
Ensure both match exactly:
```properties
AttributesToCSV:
  Attribute List: event_timestamp,geolocation,depth,magnitude,id,place,type

MergeContent:
  Header: event_timestamp,geolocation,depth,magnitude,id,place,type
```

## Best Practices

### 1. Always Set Meaningful Filenames

❌ Bad:
```
default_filename.csv
```

✅ Good:
```
earthquakes_merged_20200207_151457.csv
data_${source}_${now():format('yyyyMMdd_HHmmss')}.csv
```

### 2. Include Headers in CSV

Always add header row for clarity:
```properties
MergeContent:
  Header: col1,col2,col3
```

### 3. Handle Nulls Explicitly

```properties
AttributesToCSV:
  Null Value: NULL
  # Or empty string, or "N/A"
```

### 4. Use Appropriate Merge Sizes

```properties
# Too small: Too many files, overhead
Minimum Number of Entries: 10

# Too large: Memory issues, long waits
Maximum Number of Entries: 10000000

# Balanced:
Minimum Number of Entries: 1000
Maximum Number of Entries: 100000
Max Bin Age: 5 min
```

### 5. Organize Output Directories

```
/output/
  ├── earthquakes/
  │   ├── 2020/
  │   │   ├── 02/
  │   │   │   ├── 07/
  │   │   │   │   └── earthquakes_merged_151457.csv
```

### 6. Add Data Quality Checks

Before writing files, validate:

**RouteOnAttribute** (before MergeContent):
```properties
# Property: has_required_fields
# Value: 
${event_timestamp:isEmpty():not():and(
  ${magnitude:isEmpty():not()}
)}
```

Route only valid records to CSV output.

### 7. Archive Source Files

After successful processing:

**PutFile** for source files:
```properties
Directory: /archive/${now():format('yyyy/MM/dd')}
```

### 8. Monitor File Creation

Add **LogAttribute** after PutFile:
```
Log Level: info
Attributes to Log: filename, absolute.path, fragment.count
```

## Real-World Scenarios

### Scenario 1: Daily Reports

**Requirement**: Create one CSV file per day with all records

**Solution**:
```properties
RouteOnAttribute (before MergeContent):
  # Group by date
  daily_group: ${event_timestamp:substring(0,10)}

MergeContent:
  Correlation Attribute Name: daily_group
  Minimum Number of Entries: 1
  Max Bin Age: 24 hours

UpdateAttribute:
  filename: earthquakes_${daily_group}.csv
  directory: /output/daily-reports
```

### Scenario 2: Partner Data Feed

**Requirement**: Create files every hour, max 50MB each

**Solution**:
```properties
MergeContent:
  Minimum Number of Entries: 1000
  Maximum Group Size: 50 MB
  Max Bin Age: 1 hour

UpdateAttribute:
  filename: partner_feed_${now():format('yyyyMMdd_HH')}00.csv

PutFile:
  Directory: /sftp/partner/incoming
```

### Scenario 3: Backup Archive

**Requirement**: Compress and archive processed data

**Solution**:
```properties
MergeContent:
  Merge Format: ZIP
  Compression Level: 9
  Maximum Number of Entries: 100000

UpdateAttribute:
  filename: archive_${now():format('yyyyMMdd')}.zip
  directory: /backup/${now():format('yyyy/MM')}

PutFile:
  Directory: ${directory}
  Create Missing Directories: true
```

## Summary

### What You Learned

✅ Convert JSON to CSV using AttributesToCSV
✅ Extract JSON fields to attributes
✅ Merge multiple records into larger files
✅ Set dynamic filenames with timestamps
✅ Write files to filesystem
✅ Handle CSV headers and formatting
✅ Optimize file creation performance

### Key Processors

- **EvaluateJsonPath**: Extract JSON to attributes
- **AttributesToCSV**: Convert attributes to CSV format
- **MergeContent**: Combine multiple flow files
- **UpdateAttribute**: Set filename and metadata
- **PutFile**: Write to filesystem

```

## 05_twitter_integration.md

```markdown
# Twitter Data Integration with Apache NiFi

## Overview

This guide demonstrates how to collect real-time Twitter data using Apache NiFi's GetTwitter processor, transform the data, and prepare it for storage or analysis.

## Important Note

⚠️ **Twitter API Changes**: As of 2023, Twitter (now X) has significantly changed its API access and pricing. The GetTwitter processor may require updates or alternative approaches. This guide provides the conceptual framework that can be adapted to current API requirements.

## Tutorial Scenario

**Goal**: Collect tweets, transform with Jolt, filter with QueryRecord, and store results

**Data Flow**:
1. Get tweets from Twitter API
2. Transform JSON structure with Jolt
3. Filter tweets with QueryRecord
4. Store in database or Elasticsearch

## Prerequisites

### Twitter Developer Account Setup

1. **Create Twitter Developer Account**:
   - Visit: https://developer.twitter.com
   - Apply for developer access
   - Create a new app

2. **Generate Credentials**:
   - Consumer API Key
   - Consumer API Secret
   - Access Token
   - Access Token Secret

3. **Copy Credentials**: Save these securely (you'll need them)

### NiFi Requirements

- NiFi 1.x or later
- Network access to Twitter API
- Sufficient storage for tweet data

## Flow Architecture

```
GetTwitter → JoltTransformJSON → QueryRecord → [Destination]
                                      ├→ FilterByKeyword
                                      ├→ FilterByDevice
                                      └→ FilterByLanguage
```

## Step 1: Create Process Group

1. **Add Process Group**: Name it "Twitter Data"
2. **Double-click** to enter
3. Begin building flow

## Step 2: Configure GetTwitter Processor

### Add GetTwitter Processor

1. **Drag processor** onto canvas
2. **Search**: "GetTwitter"
3. **Add**: GetTwitter

### Twitter API Credentials

1. **Double-click** GetTwitter processor
2. **Navigate to** Properties tab

#### Required Properties

```properties
Twitter Endpoint: Sample Endpoint
Consumer Key: [Your Consumer API Key]
Consumer Secret: [Your Consumer API Secret]
Access Token: [Your Access Token]
Access Token Secret: [Your Access Token Secret]
```

#### Twitter Endpoint Options

**Sample Endpoint** (Recommended for learning):
- Random 1% sample of all tweets
- No filtering
- Good for testing
- High volume

**Filter Endpoint**:
- Filter by keywords, users, locations
- More targeted data
- Requires additional configuration

**Firehose Endpoint**:
- ALL public tweets
- Requires special access
- Very high volume
- Enterprise only

### Optional Filtering (Filter Endpoint Only)

If using Filter Endpoint:

```properties
Languages: en,es,fr                    # Comma-separated language codes
Terms to Filter On: earthquake,tsunami  # Keywords to track
IDs to Follow: 12345,67890             # User IDs to follow
Locations to Filter On: -74,40,-73,41  # Bounding box (SW long,lat,NE long,lat)
```

**Location Format**:
```
Southwest Corner: longitude,latitude
Northeast Corner: longitude,latitude
Example (New York): -74.0,40.7,-73.9,40.8
```

### Scheduling

```properties
Run Schedule: 0 sec (continuous)
Execution: Primary Node (if clustered)
```

### Settings

```
Automatically Terminate Relationships:
☐ success (connect to next processor)
```

### Apply and Test

1. **Click Apply**
2. **Start processor**
3. **Observe queue** filling with tweets
4. **Stop after** collecting sample (50-100 tweets)

## Understanding Tweet JSON Structure

### Sample Tweet JSON

```json
{
  "created_at": "Fri Feb 07 15:30:00 +0000 2020",
  "id": 1225453839171543040,
  "id_str": "1225453839171543040",
  "text": "This is a sample tweet about earthquakes #earthquake",
  "source": "<a href=\"http://twitter.com/download/iphone\" rel=\"nofollow\">Twitter for iPhone</a>",
  "truncated": false,
  "in_reply_to_status_id": null,
  "in_reply_to_status_id_str": null,
  "in_reply_to_user_id": null,
  "in_reply_to_user_id_str": null,
  "in_reply_to_screen_name": null,
  "user": {
    "id": 123456789,
    "id_str": "123456789",
    "name": "John Doe",
    "screen_name": "johndoe",
    "location": "San Francisco, CA",
    "description": "Just a regular person tweeting",
    "url": null,
    "protected": false,
    "followers_count": 1000,
    "friends_count": 500,
    "listed_count": 10,
    "created_at": "Mon Jan 01 00:00:00 +0000 2015",
    "favourites_count": 2000,
    "verified": false,
    "statuses_count": 5000,
    "lang": null,
    "geo_enabled": true
  },
  "geo": null,
  "coordinates": null,
  "place": null,
  "contributors": null,
  "is_quote_status": false,
  "retweet_count": 5,
  "favorite_count": 10,
  "favorited": false,
  "retweeted": false,
  "lang": "en"
}
```

### Key Fields

| Field | Description | Example |
|-------|-------------|---------|
| `created_at` | Tweet timestamp | "Fri Feb 07 15:30:00 +0000 2020" |
| `id_str` | Tweet ID (string) | "1225453839171543040" |
| `text` | Tweet content | "This is a sample tweet..." |
| `source` | Client used | "Twitter for iPhone" |
| `user.name` | User's display name | "John Doe" |
| `user.screen_name` | Username | "johndoe" |
| `user.location` | User's location | "San Francisco, CA" |
| `user.followers_count` | Follower count | 1000 |
| `user.friends_count` | Following count | 500 |
| `retweet_count` | Retweet count | 5 |
| `favorite_count` | Like count | 10 |
| `lang` | Language code | "en" |

## Step 3: Transform with Jolt

### Why Transform?

Original tweet JSON has **50+ fields**. We typically only need:
- Tweet ID
- Tweet text
- Timestamp
- User information
- Engagement metrics
- Source device

### Add JoltTransformJSON Processor

1. **Add processor**: JoltTransformJSON
2. **Rename**: "Transform Twitter JSON"
3. **Connect**: GetTwitter (success) → JoltTransformJSON

### Jolt Specification

```json
[
  {
    "operation": "shift",
    "spec": {
      "created_at": "created_at",
      "id_str": "tweet_id",
      "text": "text",
      "source": "source",
      "user": {
        "id_str": "user.id",
        "name": "user.name",
        "screen_name": "user.screen_name",
        "location": "user.location",
        "followers_count": "user.followers_count",
        "friends_count": "user.friends_count"
      },
      "retweet_count": "retweets",
      "favorite_count": "likes"
    }
  }
]
```

### Configuration

```properties
Jolt Transformation DSL: jolt-transform-chain
Jolt Specification: [paste spec above]
```

### Settings

```
Automatically Terminate Relationships:
☑ failure
```

### Testing Jolt

1. **Get sample tweet** from queue
2. **Use Advanced Editor**:
   - Click Advanced in Jolt Specification
   - Paste sample tweet in Input
   - Paste spec in Spec
   - Click Transform
   - Verify output

**Expected Output**:
```json
{
  "created_at": "Fri Feb 07 15:30:00 +0000 2020",
  "tweet_id": "1225453839171543040",
  "text": "This is a sample tweet about earthquakes #earthquake",
  "source": "<a href=\"http://twitter.com/download/iphone\" rel=\"nofollow\">Twitter for iPhone</a>",
  "user": {
    "id": "123456789",
    "name": "John Doe",
    "screen_name": "johndoe",
    "location": "San Francisco, CA",
    "followers_count": 1000,
    "friends_count": 500
  },
  "retweets": 5,
  "likes": 10
}
```

### Extract Device from Source

The `source` field contains HTML. Let's clean it:

**Additional Jolt Operation**:
```json
[
  {
    "operation": "shift",
    "spec": {
      "created_at": "created_at",
      "id_str": "tweet_id",
      "text": "text",
      "source": "source",
      "user": {
        "id_str": "user_id",
        "name": "name",
        "screen_name": "screen_name",
        "location": "location",
        "followers_count": "follower_count",
        "friends_count": "friend_count"
      },
      "retweet_count": "retweets",
      "favorite_count": "likes"
    }
  }
]
```

**Note**: Extracting device name from HTML `source` field is complex in Jolt. Better to use QueryRecord or ReplaceText for this.

## Step 4: Filter with QueryRecord

### Purpose

Filter tweets based on criteria:
- Keyword filtering
- Device filtering
- Language filtering
- User criteria

### Add QueryRecord Processor

1. **Add processor**: QueryRecord
2. **Rename**: "Filter Tweets"
3. **Connect**: JoltTransformJSON (success) → QueryRecord

### Configure Controller Services

#### JSON Tree Reader

1. **Click Configuration** (canvas background)
2. **Controller Services** tab
3. **Add Service**: JsonTreeReader
4. **Name**: "JSON Reader - Twitter"
5. **Properties**: Use defaults
6. **Enable**

#### JSON Record Set Writer

1. **Add Service**: JsonRecordSetWriter
2. **Name**: "JSON Writer - Twitter"
3. **Properties**:
   ```properties
   Output Grouping: One Line Per Object
   ```
4. **Enable**

### QueryRecord Configuration

```properties
Record Reader: JSON Reader - Twitter
Record Writer: JSON Writer - Twitter
Include Zero Record FlowFiles: false
Cache Schema: false
```

### Create Queries

#### Query 1: Keyword Filter (Earthquakes)

**Property Name**: `earthquakes`

**SQL Query**:
```sql
SELECT *
FROM FLOWFILE
WHERE LOWER(text) LIKE '%earthquake%'
   OR LOWER(text) LIKE '%seismic%'
   OR LOWER(text) LIKE '%tremor%'
```

**Explanation**:
- Searches for earthquake-related keywords
- Case-insensitive (`LOWER()`)
- Multiple keyword options

#### Query 2: Device Filter (iPhone)

**Property Name**: `iphone_tweets`

**SQL Query**:
```sql
SELECT *,
       LOWER(source) AS source_lower
FROM FLOWFILE
WHERE LOWER(source) LIKE '%iphone%'
```

#### Query 3: Device Filter (Android)

**Property Name**: `android_tweets`

**SQL Query**:
```sql
SELECT *
FROM FLOWFILE
WHERE LOWER(source) LIKE '%android%'
```

#### Query 4: High Engagement

**Property Name**: `viral_tweets`

**SQL Query**:
```sql
SELECT *,
       (CAST(retweets AS INT) + CAST(likes AS INT)) AS total_engagement
FROM FLOWFILE
WHERE CAST(retweets AS INT) > 100
   OR CAST(likes AS INT) > 100
```

#### Query 5: Verified Users with Followers

**Property Name**: `influencer_tweets`

**SQL Query**:
```sql
SELECT *
FROM FLOWFILE
WHERE CAST(follower_count AS INT) > 10000
```

### Connect Outputs

Each query creates a relationship. Connect as needed:

```
QueryRecord (earthquakes) → [Process earthquake tweets]
QueryRecord (iphone_tweets) → [Process iPhone tweets]
QueryRecord (android_tweets) → [Process Android tweets]
QueryRecord (viral_tweets) → [Process viral tweets]
QueryRecord (influencer_tweets) → [Process influencer tweets]
```

### Settings

```
Automatically Terminate Relationships:
☑ failure
☑ original
☐ earthquakes (connect to next processor)
☐ iphone_tweets (connect to next processor)
☐ android_tweets (connect to next processor)
☑ viral_tweets (or connect)
☑ influencer_tweets (or connect)
```

## Advanced QueryRecord Patterns

### Adding Calculated Fields

```sql
SELECT *,
       CAST(follower_count AS INT) / CAST(friend_count AS INT) AS follower_ratio,
       LENGTH(text) AS tweet_length,
       CASE 
         WHEN CAST(retweets AS INT) > 1000 THEN 'viral'
         WHEN CAST(retweets AS INT) > 100 THEN 'popular'
         ELSE 'normal'
       END AS popularity_tier
FROM FLOWFILE
```

### Date/Time Manipulation

```sql
SELECT *,
       SUBSTRING(created_at, 1, 10) AS date_only,
       SUBSTRING(created_at, 12, 8) AS time_only
FROM FLOWFILE
WHERE created_at IS NOT NULL
```

### Filtering by Multiple Conditions

```sql
SELECT *
FROM FLOWFILE
WHERE (LOWER(text) LIKE '%earthquake%' 
       OR LOWER(text) LIKE '%tsunami%')
  AND CAST(follower_count AS INT) > 1000
  AND location IS NOT NULL
  AND location != ''
```

### Deduplication

```sql
SELECT DISTINCT tweet_id, text, created_at, user_id
FROM FLOWFILE
```

## Step 5: Store Results

### Option 1: Store in MySQL

```
QueryRecord → ConvertJSONtoSQL → PutSQL
```

**Table Schema**:
```sql
CREATE TABLE tweets (
    tweet_id VARCHAR(100) PRIMARY KEY,
    created_at VARCHAR(100),
    text TEXT,
    source VARCHAR(255),
    user_id VARCHAR(100),
    name VARCHAR(255),
    screen_name VARCHAR(100),
    location VARCHAR(255),
    follower_count INT,
    friend_count INT,
    retweets INT,
    likes INT,
    collected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Option 2: Store in Elasticsearch

```
QueryRecord → PutElasticsearchHttp
```

**Index Mapping**:
```json
{
  "mappings": {
    "properties": {
      "tweet_id": {"type": "keyword"},
      "created_at": {"type": "date", "format": "EEE MMM dd HH:mm:ss Z yyyy"},
      "text": {"type": "text"},
      "source": {"type": "keyword"},
      "user_id": {"type": "keyword"},
      "name": {"type": "text"},
      "screen_name": {"type": "keyword"},
      "location": {"type": "text"},
      "follower_count": {"type": "integer"},
      "friend_count": {"type": "integer"},
      "retweets": {"type": "integer"},
      "likes": {"type": "integer"}
    }
  }
}
```

### Option 3: Store as JSON Files

```
QueryRecord → UpdateAttribute → PutFile
```

**UpdateAttribute**:
```properties
filename: tweets_${now():format('yyyyMMdd_HHmmss')}.json
directory: /data/twitter/${now():format('yyyy/MM/dd')}
```

## Complete Flow Example

### Scenario: Track Earthquake Tweets by Device

```
┌──────────────┐
│  GetTwitter  │
│   (Sample)   │
└──────┬───────┘
       │
┌──────▼───────────┐
│ JoltTransform    │
│ (Clean fields)   │
└──────┬───────────┘
       │
┌──────▼───────────┐
│  QueryRecord     │
│  (Filter)        │
└──┬───┬───┬───────┘
   │   │   │
   │   │   └─────────────────┐
   │   │                     │
   │   │            ┌────────▼─────────┐
   │   │            │ Android Quakes   │
   │   │            │ (android+quake)  │
   │   │            └────────┬─────────┘
   │   │                     │
   │   │            ┌────────▼─────────┐
   │   │            │  PutElasticsearch│
   │   │            │  (android-index) │
   │   │            └──────────────────┘
   │   │
   │   └──────────────────────┐
   │                          │
   │                 ┌────────▼─────────┐
   │                 │  iPhone Quakes   │
   │                 │ (iphone+quake)   │
   │                 └────────┬─────────┘
   │                          │
   │                 ┌────────▼─────────┐
   │                 │ PutElasticsearch │
   │                 │  (iphone-index)  │
   │                 └──────────────────┘
   │
   └──────────────────────────┐
                              │
                     ┌────────▼─────────┐
                     │   All Quakes     │
                     │ (all devices)    │
                     └────────┬─────────┘
                              │
                     ┌────────▼─────────┐
                     │  PutElasticsearch│
                     │   (all-index)    │
                     └──────────────────┘
```

### QueryRecord Queries

**Query 1 - Android Earthquakes**:
```sql
SELECT *
FROM FLOWFILE
WHERE LOWER(source) LIKE '%android%'
  AND (LOWER(text) LIKE '%earthquake%'
       OR LOWER(text) LIKE '%seismic%')
```

**Query 2 - iPhone Earthquakes**:
```sql
SELECT *
FROM FLOWFILE
WHERE LOWER(source) LIKE '%iphone%'
  AND (LOWER(text) LIKE '%earthquake%'
       OR LOWER(text) LIKE '%seismic%')
```

**Query 3 - All Earthquakes**:
```sql
SELECT *
FROM FLOWFILE
WHERE LOWER(text) LIKE '%earthquake%'
   OR LOWER(text) LIKE '%seismic%'
```

## Data Analysis Examples

### Elasticsearch Queries

Once data is in Elasticsearch, you can analyze:

**Top Hashtags**:
```json
{
  "size": 0,
  "aggs": {
    "hashtags": {
      "terms": {
        "field": "text.keyword",
        "include": "#.*"
      }
    }
  }
}
```

**Tweets Over Time**:
```json
{
  "size": 0,
  "aggs": {
    "tweets_over_time": {
      "date_histogram": {
        "field": "created_at",
        "interval": "hour"
      }
    }
  }
}
```

**Top Users**:
```json
{
  "size": 0,
  "aggs": {
    "top_users": {
      "terms": {
        "field": "screen_name.keyword",
        "size": 10,
        "order": {"_count": "desc"}
      }
    }
  }
}
```

## Performance Optimization

### Rate Limiting

Twitter API has rate limits. Monitor and respect them:

**GetTwitter Settings**:
```properties
Run Schedule: 1 sec (don't overload API)
```

### Backpressure

Prevent memory issues:

**Connection Settings** (right-click connection):
```properties
Object Threshold: 10000
Size Threshold: 100 MB
```

### Concurrent Processing

```properties
JoltTransformJSON:
  Concurrent Tasks: 5

QueryRecord:
  Concurrent Tasks: 5
```

### Batch Writing

For database inserts:

```properties
PutSQL:
  Batch Size: 1000
```

## Monitoring and Alerting

### Add Monitoring Attributes

**UpdateAttribute** (before storage):
```properties
processing_timestamp: ${now():format('yyyy-MM-dd HH:mm:ss')}
flow_version: 1.0
data_source: twitter_api
```

### Alert on Errors

**RouteOnAttribute** (after GetTwitter):
```properties
# Route errors to alert
has_error: ${error.message:isEmpty():not()}
```

Connect `has_error` to **PutEmail** or notification service.

### Track Metrics

**LogAttribute** (periodically):
```properties
Log Level: INFO
Attributes to Log: fragment.count, processing.time
Log Payload: false
```

## Troubleshooting

### Issue 1: No Tweets Appearing

**Symptoms**: GetTwitter runs but no flow files

**Causes**:
1. Invalid API credentials
2. Rate limit exceeded
3. Network connectivity
4. Endpoint not accessible

**Solutions**:
```
1. Verify credentials in Twitter Developer Portal
2. Check NiFi logs for API errors
3. Test network connectivity to api.twitter.com
4. Ensure firewall allows HTTPS (443) outbound
5. Check Twitter API status page
```

### Issue 2: Jolt Transformation Fails

**Symptoms**: Failure relationship has flow files

**Causes**:
- JSON structure doesn't match spec
- Invalid Jolt syntax

**Solutions**:
1. List failure queue
2. View flow file content
3. Verify JSON structure
4. Test spec in Jolt editor
5. Update spec to match actual structure

### Issue 3: QueryRecord No Matches

**Symptoms**: All queries return zero records

**Causes**:
- SQL syntax errors
- Field names don't match
- Data type mismatches

**Solutions**:
```sql
-- Test with select all first
SELECT * FROM FLOWFILE

-- Then add filters incrementally
SELECT * FROM FLOWFILE WHERE text IS NOT NULL
SELECT * FROM FLOWFILE WHERE LOWER(text) LIKE '%test%'
```

### Issue 4: High Memory Usage

**Symptoms**: NiFi runs out of memory

**Causes**:
- Too many tweets in queues
- No backpressure
- Large tweet content

**Solutions**:
1. Add backpressure thresholds
2. Increase NiFi heap size
3. Add flow control processors
4. Reduce concurrent tasks

## Best Practices

### 1. Respect API Limits

```properties
GetTwitter:
  Run Schedule: 1 sec minimum
```

Monitor your usage in Twitter Developer Portal.

### 2. Handle Errors Gracefully

Always connect failure relationships:
```
Processor (failure) → LogAttribute → [Archive or Alert]
```

### 3. Store Raw Data

Keep original tweets before transformation:
```
GetTwitter → [Branch 1: Transform] 
          └→ [Branch 2: Archive Raw]
```

### 4. Add Timestamps

Track when data was collected:
```properties
UpdateAttribute:
  collected_at: ${now():format('yyyy-MM-dd HH:mm:ss')}
```

### 5. Version Your Flows

Use ProcessGroup variables:
```properties
flow.version: 1.0
flow.author: your_name
flow.updated: 2024-01-15
```

### 6. Monitor Data Quality

Add validation:
```sql
SELECT * FROM FLOWFILE
WHERE tweet_id IS NOT NULL
  AND text IS NOT NULL
  AND LENGTH(text) > 0
```

### 7. Implement Deduplication

Prevent duplicate tweets:
```sql
-- In QueryRecord
SELECT DISTINCT tweet_id, * FROM FLOWFILE
```

Or use **DetectDuplicate** processor.

## Alternative Approaches

### Using Twitter v2 API

Twitter v2 API requires different authentication:

**Use InvokeHTTP Instead**:
```properties
HTTP Method: GET
Remote URL: https://api.twitter.com/2/tweets/search/recent
Authorization: Bearer Token
```

**Bearer Token** (from Twitter Developer Portal)

### Real-time Streaming with ConsumeKafka

If Twitter data is streamed to Kafka:

```
ConsumeKafka → JoltTransform → QueryRecord → [Destination]
```

### Using Third-Party Services

Services like:
- Brandwatch
- Sprinklr
- Hootsuite

Provide enhanced APIs with better filtering.

## Summary

### What You Learned

✅ Configure GetTwitter processor
✅ Transform tweet JSON with Jolt
✅ Filter tweets with QueryRecord
✅ Create multiple routing paths
✅ Store data in various destinations
✅ Handle Twitter API limitations
✅ Monitor and troubleshoot flows

### Key Processors

- **GetTwitter**: Collect tweets from API
- **JoltTransformJSON**: Clean and reshape data
- **QueryRecord**: Filter with SQL queries
- **RouteOnAttribute**: Branch based on criteria
- **PutElasticsearch/PutSQL**: Store results

```

I'll search for our previous conversation to see where we left off.Perfect! I found our conversation. We completed guides 01-05 and I'll now continue with the remaining guides 06-14. Let me pick up where we left off:

## 06_query_record.md

```markdown
# Query Record Processor - Advanced SQL on FlowFiles

## Overview

The QueryRecord processor is one of NiFi's most powerful data processing components. It allows you to execute SQL queries against FlowFiles without loading data into a database, supporting complex filtering, aggregation, joins, and transformations.

## Core Concepts

### What is QueryRecord?

- **SQL Interface**: Use standard SQL syntax on streaming data
- **In-Memory Processing**: No database required
- **Schema-Aware**: Works with structured data (JSON, Avro, CSV, XML)
- **Multiple Outputs**: Route results to different relationships

### When to Use QueryRecord

✅ **Use QueryRecord when you need:**
- Complex filtering logic
- Data aggregation and grouping
- Joins between FlowFiles
- Field calculations and transformations
- Multiple output paths based on SQL predicates

❌ **Avoid QueryRecord for:**
- Simple attribute routing (use RouteOnAttribute)
- Single field extraction (use EvaluateJsonPath)
- Large-scale aggregations across millions of records (consider database)

## Configuration

### 1. Record Reader

Choose based on input format:

| Format | Controller Service |
|--------|-------------------|
| JSON | JsonTreeReader |
| CSV | CSVReader |
| Avro | AvroReader |
| XML | XMLReader |

**JsonTreeReader Configuration:**
```properties
Schema Access Strategy: Infer Schema
```

### 2. Record Writer

Choose based on desired output:

| Format | Controller Service |
|--------|-------------------|
| JSON | JsonRecordSetWriter |
| CSV | CSVRecordSetWriter |
| Avro | AvroRecordSetWriter |

**JsonRecordSetWriter Configuration:**
```properties
Schema Write Strategy: Inherit Record Schema
Schema Access Strategy: Inherit Record Schema
Pretty Print JSON: true
Suppress Null Values: Never Suppress
```

### 3. Query Configuration

Add dynamic properties for each query:

```properties
Property Name: [relationship_name]
Property Value: [SQL query]
```

## SQL Syntax and Examples

### Basic SELECT Statements

**Filter records:**
```sql
-- Property: filtered
SELECT * FROM FLOWFILE 
WHERE status = 'active' 
  AND age >= 18
```

**Select specific fields:**
```sql
-- Property: simplified
SELECT 
  id,
  first_name,
  last_name,
  email
FROM FLOWFILE
```

**Calculate fields:**
```sql
-- Property: calculated
SELECT 
  *,
  price * quantity AS total,
  CASE 
    WHEN quantity > 100 THEN 'bulk'
    WHEN quantity > 10 THEN 'medium'
    ELSE 'small'
  END AS order_size
FROM FLOWFILE
```

### Aggregations

**Group and count:**
```sql
-- Property: summary
SELECT 
  category,
  COUNT(*) AS record_count,
  SUM(amount) AS total_amount,
  AVG(amount) AS avg_amount,
  MIN(amount) AS min_amount,
  MAX(amount) AS max_amount
FROM FLOWFILE
GROUP BY category
```

**Filter aggregated results:**
```sql
-- Property: high_volume
SELECT 
  customer_id,
  COUNT(*) AS order_count,
  SUM(total) AS total_spent
FROM FLOWFILE
GROUP BY customer_id
HAVING COUNT(*) > 10
```

### String Operations

```sql
SELECT 
  UPPER(first_name) AS first_name_upper,
  LOWER(email) AS email_lower,
  CONCAT(first_name, ' ', last_name) AS full_name,
  SUBSTRING(phone, 1, 3) AS area_code,
  TRIM(address) AS clean_address,
  LENGTH(description) AS desc_length
FROM FLOWFILE
```

### Date/Time Functions

```sql
SELECT 
  *,
  CURRENT_TIMESTAMP AS processed_at,
  EXTRACT(YEAR FROM order_date) AS order_year,
  EXTRACT(MONTH FROM order_date) AS order_month,
  DATEDIFF(DAY, order_date, ship_date) AS days_to_ship
FROM FLOWFILE
```

### CASE Expressions

```sql
SELECT 
  *,
  CASE 
    WHEN score >= 90 THEN 'A'
    WHEN score >= 80 THEN 'B'
    WHEN score >= 70 THEN 'C'
    WHEN score >= 60 THEN 'D'
    ELSE 'F'
  END AS grade,
  CASE
    WHEN amount > 1000 THEN amount * 0.1
    WHEN amount > 500 THEN amount * 0.05
    ELSE 0
  END AS discount
FROM FLOWFILE
```

### NULL Handling

```sql
SELECT 
  COALESCE(phone, 'N/A') AS phone,
  COALESCE(middle_name, '') AS middle_name,
  NULLIF(status, 'unknown') AS status,
  CASE WHEN email IS NULL THEN 'missing' ELSE 'present' END AS email_status
FROM FLOWFILE
WHERE city IS NOT NULL
```

## Advanced Patterns

### Multiple Output Relationships

Create different outputs from same input:

```sql
-- High value customers
-- Property: high_value
SELECT * FROM FLOWFILE 
WHERE total_purchases > 10000

-- Medium value customers
-- Property: medium_value
SELECT * FROM FLOWFILE 
WHERE total_purchases BETWEEN 1000 AND 10000

-- Low value customers
-- Property: low_value
SELECT * FROM FLOWFILE 
WHERE total_purchases < 1000

-- All customers summary
-- Property: summary
SELECT 
  CASE 
    WHEN total_purchases > 10000 THEN 'high'
    WHEN total_purchases >= 1000 THEN 'medium'
    ELSE 'low'
  END AS segment,
  COUNT(*) AS customer_count,
  AVG(total_purchases) AS avg_purchases
FROM FLOWFILE
GROUP BY 
  CASE 
    WHEN total_purchases > 10000 THEN 'high'
    WHEN total_purchases >= 1000 THEN 'medium'
    ELSE 'low'
  END
```

### Data Quality Checks

```sql
-- Property: valid_records
SELECT * FROM FLOWFILE
WHERE email LIKE '%@%.%'
  AND phone IS NOT NULL
  AND LENGTH(phone) >= 10
  AND age BETWEEN 18 AND 120
  AND zip_code ~ '^[0-9]{5}$'

-- Property: invalid_records
SELECT 
  *,
  CASE 
    WHEN email NOT LIKE '%@%.%' THEN 'Invalid email'
    WHEN phone IS NULL THEN 'Missing phone'
    WHEN LENGTH(phone) < 10 THEN 'Invalid phone'
    WHEN age NOT BETWEEN 18 AND 120 THEN 'Invalid age'
    ELSE 'Unknown error'
  END AS validation_error
FROM FLOWFILE
WHERE email NOT LIKE '%@%.%'
   OR phone IS NULL
   OR LENGTH(phone) < 10
   OR age NOT BETWEEN 18 AND 120
```

### Deduplication

```sql
-- Keep only first occurrence
-- Property: unique
SELECT DISTINCT 
  email,
  MIN(created_date) AS first_seen,
  COUNT(*) AS occurrence_count
FROM FLOWFILE
GROUP BY email

-- Find duplicates
-- Property: duplicates
SELECT 
  email,
  COUNT(*) AS count
FROM FLOWFILE
GROUP BY email
HAVING COUNT(*) > 1
```

### Pivoting Data

```sql
-- Transform rows to columns
SELECT 
  customer_id,
  MAX(CASE WHEN product = 'ProductA' THEN quantity ELSE 0 END) AS product_a_qty,
  MAX(CASE WHEN product = 'ProductB' THEN quantity ELSE 0 END) AS product_b_qty,
  MAX(CASE WHEN product = 'ProductC' THEN quantity ELSE 0 END) AS product_c_qty
FROM FLOWFILE
GROUP BY customer_id
```

### Window Functions (Calcite SQL)

```sql
-- Running totals
SELECT 
  order_date,
  amount,
  SUM(amount) OVER (ORDER BY order_date) AS running_total,
  AVG(amount) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7day
FROM FLOWFILE

-- Ranking
SELECT 
  *,
  ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rank_in_category,
  DENSE_RANK() OVER (ORDER BY sales DESC) AS overall_rank
FROM FLOWFILE
```

## Common Use Cases

### Use Case 1: E-commerce Order Processing

**Scenario**: Filter and categorize orders

```sql
-- High priority orders (Property: priority)
SELECT 
  *,
  'PRIORITY' AS processing_flag
FROM FLOWFILE
WHERE total > 500 
  OR customer_tier = 'premium'
  OR shipping_method = 'express'

-- Standard orders (Property: standard)
SELECT 
  *,
  'STANDARD' AS processing_flag
FROM FLOWFILE
WHERE total <= 500 
  AND customer_tier != 'premium'
  AND shipping_method != 'express'

-- Order summary by region (Property: regional_summary)
SELECT 
  region,
  COUNT(*) AS order_count,
  SUM(total) AS total_revenue,
  AVG(total) AS avg_order_value,
  SUM(CASE WHEN total > 500 THEN 1 ELSE 0 END) AS high_value_count
FROM FLOWFILE
GROUP BY region
```

### Use Case 2: Log Analysis

**Scenario**: Parse and analyze application logs

```sql
-- Error logs (Property: errors)
SELECT * FROM FLOWFILE
WHERE log_level IN ('ERROR', 'FATAL')

-- Critical errors needing immediate attention (Property: critical)
SELECT * FROM FLOWFILE
WHERE log_level = 'FATAL'
   OR (log_level = 'ERROR' AND message LIKE '%OutOfMemory%')
   OR (log_level = 'ERROR' AND message LIKE '%Connection%')

-- Error summary by service (Property: error_summary)
SELECT 
  service_name,
  log_level,
  COUNT(*) AS error_count,
  MIN(timestamp) AS first_occurrence,
  MAX(timestamp) AS last_occurrence
FROM FLOWFILE
WHERE log_level IN ('ERROR', 'FATAL')
GROUP BY service_name, log_level
ORDER BY error_count DESC
```

### Use Case 3: Customer Data Enrichment

**Scenario**: Add calculated fields and segments

```sql
-- Property: enriched_customers
SELECT 
  *,
  DATEDIFF(DAY, registration_date, CURRENT_DATE) AS days_since_registration,
  CASE 
    WHEN total_purchases > 10000 THEN 'VIP'
    WHEN total_purchases > 5000 THEN 'Gold'
    WHEN total_purchases > 1000 THEN 'Silver'
    ELSE 'Bronze'
  END AS loyalty_tier,
  CASE 
    WHEN last_purchase_date > CURRENT_DATE - INTERVAL '30' DAY THEN 'Active'
    WHEN last_purchase_date > CURRENT_DATE - INTERVAL '90' DAY THEN 'At Risk'
    ELSE 'Inactive'
  END AS engagement_status,
  ROUND(total_purchases / NULLIF(purchase_count, 0), 2) AS avg_order_value
FROM FLOWFILE
```

### Use Case 4: IoT Sensor Data

**Scenario**: Filter anomalies and aggregate metrics

```sql
-- Anomalies (Property: anomalies)
SELECT 
  *,
  'TEMPERATURE_HIGH' AS anomaly_type
FROM FLOWFILE
WHERE sensor_type = 'temperature' 
  AND value > 100

UNION ALL

SELECT 
  *,
  'TEMPERATURE_LOW' AS anomaly_type
FROM FLOWFILE
WHERE sensor_type = 'temperature' 
  AND value < 0

UNION ALL

SELECT 
  *,
  'PRESSURE_HIGH' AS anomaly_type
FROM FLOWFILE
WHERE sensor_type = 'pressure' 
  AND value > 150

-- Aggregated metrics (Property: metrics)
SELECT 
  sensor_id,
  sensor_type,
  COUNT(*) AS reading_count,
  AVG(value) AS avg_value,
  MIN(value) AS min_value,
  MAX(value) AS max_value,
  STDDEV(value) AS std_dev
FROM FLOWFILE
GROUP BY sensor_id, sensor_type
```

## Performance Considerations

### 1. Schema Inference vs. Explicit Schema

**Schema Inference** (easier but slower):
```properties
JsonTreeReader:
  Schema Access Strategy: Infer Schema
```

**Explicit Schema** (faster for large datasets):
```properties
JsonTreeReader:
  Schema Access Strategy: Use 'Schema Text' Property
  Schema Text: {
    "type": "record",
    "name": "Customer",
    "fields": [
      {"name": "id", "type": "int"},
      {"name": "name", "type": "string"},
      {"name": "email", "type": "string"}
    ]
  }
```

### 2. Record Batching

```properties
QueryRecord:
  Include Zero Record FlowFiles: false
```

Combine with MergeRecord before QueryRecord for better performance:
```
MergeRecord (1000 records) → QueryRecord
```

### 3. Query Complexity

**Simple queries** (fast):
```sql
SELECT * FROM FLOWFILE WHERE status = 'active'
```

**Complex aggregations** (slower):
```sql
SELECT 
  category,
  subcategory,
  COUNT(DISTINCT customer_id),
  AVG(amount),
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY amount)
FROM FLOWFILE
GROUP BY category, subcategory
```

For very complex aggregations, consider:
- Breaking into multiple QueryRecord processors
- Using database for aggregation
- Implementing custom processors

### 4. Memory Management

Monitor heap usage for large FlowFiles:
```
nifi.properties:
  java.arg.2=-Xms2g
  java.arg.3=-Xmx4g
```

## Error Handling

### Common Errors and Solutions

**Error: "Unable to parse"**
```
Solution: Check Record Reader configuration and schema
- Verify JSON is valid
- Check Schema Access Strategy
- Review sample data format
```

**Error: "Column not found"**
```
Solution: Verify field names in SQL
- Use exact case-sensitive names
- Check nested field syntax: parent.child
- Review inferred schema
```

**Error: "Out of Memory"**
```
Solution: Reduce batch size or increase heap
- Use MergeRecord to control batch size
- Split large files before QueryRecord
- Increase NiFi JVM heap size
```

### Validation Flow

```
GenerateFlowFile (test data)
  ↓
QueryRecord
  ↓ original
LogAttribute (verify all records passed through)
  ↓ [query_name]
LogAttribute (verify query results)
```

## Integration Patterns

### Pattern 1: Multi-Stage Filtering

```
GetFile
  ↓
QueryRecord #1 (Initial Filter)
  ↓ valid
QueryRecord #2 (Categorization)
  ↓ category_a / category_b / category_c
[Different destinations]
```

### Pattern 2: Quality Assurance Pipeline

```
Source
  ↓
QueryRecord (Validation)
  ↓ valid          ↓ invalid
PutDatabase    LogAttribute
                     ↓
               PutFile (quarantine)
```

### Pattern 3: Aggregation and Distribution

```
ConsumeKafka
  ↓
MergeRecord (batch 1000)
  ↓
QueryRecord (aggregate)
  ↓ summary
RouteOnAttribute
  ↓ different routes
[Multiple destinations based on attributes]
```

## Comparison with Other Processors

### QueryRecord vs. RouteOnAttribute

**Use RouteOnAttribute when:**
- Simple attribute-based routing
- No data transformation needed
- Checking FlowFile attributes (not content)

```
RouteOnAttribute:
  ${file.size:gt(1000000)}
```

**Use QueryRecord when:**
- Complex logic on record content
- Multiple fields involved
- Data transformation needed

### QueryRecord vs. JoltTransformJSON

**Use JoltTransformJSON when:**
- Restructuring JSON
- Renaming fields
- Nested transformations

**Use QueryRecord when:**
- Filtering records
- Aggregations
- SQL-style operations

**Combine both:**
```
JoltTransformJSON (reshape)
  ↓
QueryRecord (filter & aggregate)
```

### QueryRecord vs. UpdateRecord

**Use UpdateRecord when:**
- Modifying specific fields
- Adding calculated fields to each record
- Conditional updates

**Use QueryRecord when:**
- Filtering entire records
- Aggregating across records
- Multiple output paths

## Testing Queries

### 1. Use GenerateFlowFile

```json
Test data:
[
  {"id": 1, "name": "Alice", "age": 30, "city": "NYC"},
  {"id": 2, "name": "Bob", "age": 25, "city": "LA"},
  {"id": 3, "name": "Charlie", "age": 35, "city": "NYC"}
]
```

### 2. Add LogAttribute Processors

```
QueryRecord
  ↓ [each relationship]
LogAttribute (Log Level: info)
```

### 3. Check Provenance

Right-click processor → View Data Provenance
- See input/output records
- Verify transformations
- Debug issues

### 4. Validate Output

```
QueryRecord
  ↓
PutFile (test output directory)
```

Review generated files to confirm results.

## Best Practices

### 1. Name Relationships Clearly

✅ Good:
```
high_value_customers
error_records
daily_summary
```

❌ Bad:
```
output1
result
data
```

### 2. Document Complex Queries

Add comments in processor configuration:
```sql
-- Filter active customers from last 30 days
-- who made purchases over $100
SELECT * FROM FLOWFILE
WHERE status = 'active'
  AND last_purchase > CURRENT_DATE - INTERVAL '30' DAY
  AND total_purchases > 100
```

### 3. Handle NULL Values

Always consider NULLs in your queries:
```sql
WHERE email IS NOT NULL
  AND TRIM(email) != ''
  AND email LIKE '%@%.%'
```

### 4. Use Appropriate Data Types

Leverage schema for type safety:
```json
{
  "name": "Order",
  "type": "record",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "amount", "type": "double"},
    {"name": "date", "type": {"type": "long", "logicalType": "timestamp-millis"}}
  ]
}
```

### 5. Monitor Performance

Add UpdateAttribute before/after:
```properties
Before QueryRecord:
  query.start: ${now()}

After QueryRecord:
  query.end: ${now()}
  query.duration: ${query.end:minus(${query.start})}
```

## Real-World Example: Complete Flow

**Scenario**: Process e-commerce orders, categorize, validate, and route

```
ConsumeKafka (orders topic)
  ↓
UpdateAttribute (add processing timestamp)
  ↓
QueryRecord
  Properties:
    - valid_orders: SELECT * FROM FLOWFILE 
                    WHERE order_id IS NOT NULL 
                      AND customer_email LIKE '%@%.%'
                      AND total > 0
    
    - high_value: SELECT * FROM FLOWFILE 
                  WHERE total >= 1000
    
    - priority_shipping: SELECT * FROM FLOWFILE 
                        WHERE shipping_method = 'express'
                           OR total >= 500
    
    - order_summary: SELECT 
                       DATE_TRUNC('day', order_date) AS order_day,
                       COUNT(*) AS order_count,
                       SUM(total) AS daily_revenue,
                       AVG(total) AS avg_order_value
                     FROM FLOWFILE
                     GROUP BY DATE_TRUNC('day', order_date)
    
    - invalid_orders: SELECT 
                       *,
                       CASE 
                         WHEN order_id IS NULL THEN 'Missing ID'
                         WHEN customer_email NOT LIKE '%@%.%' THEN 'Invalid email'
                         WHEN total <= 0 THEN 'Invalid amount'
                       END AS error_reason
                      FROM FLOWFILE
                      WHERE order_id IS NULL
                         OR customer_email NOT LIKE '%@%.%'
                         OR total <= 0
  
  ↓ valid_orders → PutDatabaseRecord (main database)
  ↓ high_value → InvokeHTTP (alert sales team)
  ↓ priority_shipping → PutKafka (expedite topic)
  ↓ order_summary → PutElasticsearch (analytics)
  ↓ invalid_orders → PutFile (error queue)
  ↓ failure → LogAttribute → PutFile (dead letter)
```

## Summary

QueryRecord is essential for:
- ✅ Complex filtering and transformation
- ✅ SQL-based data manipulation
- ✅ Multiple output routing
- ✅ In-flight aggregations
- ✅ Data quality validation

Key Takeaways:
1. Use SQL knowledge on streaming data
2. Multiple outputs from single input
3. Combine with other processors for complex workflows
4. Monitor performance for large datasets
5. Test thoroughly with sample data

## Next Steps

- Learn [Database Connections](09_database_connections.md)
- Explore [XML Processors](08_xml_processors.md)
- Try [UpdateRecord](updaterecord_guide.md) for field modifications
```

## 07_websocket.md

```markdown
# WebSocket Integration in Apache NiFi

## Overview

WebSocket support in Apache NiFi enables real-time, bidirectional communication between NiFi and external systems. This guide covers both consuming data from WebSocket servers and creating WebSocket servers within NiFi.

## What are WebSockets?

### WebSocket vs HTTP

| Feature | HTTP | WebSocket |
|---------|------|-----------|
| Connection | Request-Response | Persistent |
| Direction | Unidirectional | Bidirectional |
| Overhead | High (headers per request) | Low (single handshake) |
| Use Case | Static content, APIs | Real-time streaming |
| Protocol | HTTP/HTTPS | WS/WSS |

### When to Use WebSockets in NiFi

✅ **Good Use Cases:**
- Real-time data feeds (stock prices, IoT sensors)
- Live notifications and alerts
- Chat applications
- Live dashboards
- Streaming telemetry data
- Game data streams

❌ **Not Recommended For:**
- Bulk file transfers (use HTTP/FTP)
- One-time API calls (use InvokeHTTP)
- Database queries (use database processors)

## WebSocket Processors

### 1. ConnectWebSocket

Establishes outbound WebSocket connections to external servers.

**Configuration:**
```properties
WebSocket Client Controller Service: [WebSocket client service]
WebSocket Client ID: ${UUID()}
```

### 2. ListenWebSocket

Creates a WebSocket server that accepts incoming connections.

**Configuration:**
```properties
WebSocket Server Controller Service: [WebSocket server service]
WebSocket Session ID: ${websocket.session.id}
```

### 3. PutWebSocket

Sends messages to connected WebSocket clients.

**Configuration:**
```properties
WebSocket Service: [WebSocket service]
WebSocket Session ID: ${websocket.session.id}
Message: [content to send]
```

## Controller Services

### JettyWebSocketClient

Connects NiFi to external WebSocket servers.

**Configuration:**
```properties
WebSocket URI: wss://stream.example.com/data
Connection Timeout: 30 sec
Session Timeout: 1 hour
Authentication: [optional]
  - Basic
  - Bearer Token
  - SSL Context Service
```

**Example URIs:**
```
wss://stream.binance.com:9443/ws/btcusdt@trade
wss://stream.twitter.com/streaming
wss://ws-feed.pro.coinbase.com
ws://localhost:8080/sensor-data
```

### JettyWebSocketServer

Creates a WebSocket server inside NiFi.

**Configuration:**
```properties
Listen Port: 8080
Input Buffer Size: 4 MB
Max Text Message Size: 1 MB
Max Binary Message Size: 10 MB
SSL Context Service: [optional for WSS]
```

**Endpoints:**
- Available at: `ws://[nifi-host]:8080`
- Secure: `wss://[nifi-host]:8443` (with SSL)

## Common Patterns

### Pattern 1: Consume External WebSocket Stream

**Use Case:** Subscribe to real-time cryptocurrency prices

```
Flow:
ConnectWebSocket
  (connects to wss://stream.binance.com)
  ↓
EvaluateJsonPath
  (extract price, timestamp, symbol)
  ↓
RouteOnAttribute
  (filter by price thresholds)
  ↓ high_price
PutElasticsearch
  (store in time-series index)
```

**ConnectWebSocket Configuration:**
```properties
WebSocket Client Service: BinanceWebSocketClient
WebSocket Client ID: crypto-stream-${UUID()}
```

**JettyWebSocketClient Configuration:**
```properties
WebSocket URI: wss://stream.binance.com:9443/ws/btcusdt@trade
Connection Timeout: 60 sec
```

**Sample Message:**
```json
{
  "e": "trade",
  "E": 1638747123456,
  "s": "BTCUSDT",
  "t": 12345,
  "p": "50000.00",
  "q": "0.001",
  "T": 1638747123450
}
```

**EvaluateJsonPath:**
```properties
symbol: $.s
price: $.p
quantity: $.q
timestamp: $.E
```

**RouteOnAttribute:**
```properties
high_price: ${price:toNumber():gt(60000)}
low_price: ${price:toNumber():lt(30000)}
normal: ${price:toNumber():ge(30000):and(${price:toNumber():le(60000)})}
```

### Pattern 2: Create WebSocket Server for IoT Devices

**Use Case:** Accept sensor data from IoT devices

```
Flow:
ListenWebSocket
  (accepts connections on port 8080)
  ↓
ValidateRecord
  (ensure valid sensor data)
  ↓ valid
RouteOnAttribute
  (route by sensor type)
  ↓ temperature / humidity / pressure
PutDatabaseRecord
  (store by sensor type)
```

**ListenWebSocket Configuration:**
```properties
WebSocket Server Service: IoTWebSocketServer
WebSocket Session ID: ${websocket.session.id}
```

**JettyWebSocketServer Configuration:**
```properties
Listen Port: 8080
Input Buffer Size: 1 MB
Max Text Message Size: 512 KB
```

**Client Connection (Python example):**
```python
import asyncio
import websockets
import json

async def send_sensor_data():
    uri = "ws://nifi-server:8080"
    async with websockets.connect(uri) as websocket:
        data = {
            "sensor_id": "temp-001",
            "type": "temperature",
            "value": 23.5,
            "unit": "celsius",
            "timestamp": "2024-01-15T10:30:00Z"
        }
        await websocket.send(json.dumps(data))
        print(f"Sent: {data}")

asyncio.run(send_sensor_data())
```

**ValidateRecord Configuration:**
```properties
Record Reader: JsonTreeReader
Record Writer: JsonRecordSetWriter
Schema:
{
  "type": "record",
  "name": "SensorData",
  "fields": [
    {"name": "sensor_id", "type": "string"},
    {"name": "type", "type": "string"},
    {"name": "value", "type": "double"},
    {"name": "unit", "type": "string"},
    {"name": "timestamp", "type": "string"}
  ]
}
```

### Pattern 3: Bidirectional Communication

**Use Case:** Send commands to devices and receive responses

```
Flow 1 (Receive):
ListenWebSocket
  ↓
LogAttribute (log incoming messages)
  ↓
RouteOnContent
  (route by message type)
  ↓ command / status / alert
[Process each type]

Flow 2 (Send):
GenerateFlowFile (scheduled commands)
  ↓
UpdateAttribute (add session ID)
  ↓
PutWebSocket
  (send to specific device)
```

**PutWebSocket Configuration:**
```properties
WebSocket Service: IoTWebSocketServer
WebSocket Session ID: device-${device.id}
Send Message Mode: Text
Message: ${command.payload}
```

**Example Command:**
```json
{
  "command": "set_threshold",
  "device_id": "temp-001",
  "parameters": {
    "threshold": 30.0,
    "alert": true
  }
}
```

### Pattern 4: Fan-Out Broadcasting

**Use Case:** Broadcast updates to all connected clients

```
Flow:
[Data Source]
  ↓
MergeContent (batch updates)
  ↓
ExecuteScript (get all sessions)
  ↓
SplitRecord (one FlowFile per session)
  ↓
PutWebSocket
  (send to each session)
```

**ExecuteScript (Groovy) - Get All Sessions:**
```groovy
import org.apache.nifi.controller.ControllerService
import org.apache.nifi.websocket.WebSocketService

def flowFile = session.get()
if (!flowFile) return

// Get WebSocket service
def webSocketService = context.controllerServiceLookup
    .getControllerService("WebSocketServerService") as WebSocketService

// Get all active sessions
def sessions = webSocketService.getActiveSessions()

// Create attributes with session list
def attributes = [:]
attributes['session.count'] = sessions.size().toString()
attributes['session.ids'] = sessions.collect { it.sessionId }.join(',')

flowFile = session.putAllAttributes(flowFile, attributes)
session.transfer(flowFile, REL_SUCCESS)
```

## Real-Time Stock Trading Example

### Complete Flow for Live Stock Data

**Architecture:**
```
External Stock API (WebSocket)
  ↓
ConnectWebSocket
  ↓
EvaluateJsonPath (parse trade data)
  ↓
QueryRecord (filter by criteria)
  ↓ significant_trades
UpdateAttribute (calculate indicators)
  ↓
RouteOnAttribute (trading signals)
  ↓ buy_signal / sell_signal / hold
PutKafka (different topics)
```

**Step 1: Connect to Stock Stream**

```properties
ConnectWebSocket:
  WebSocket Client Service: AlpacaWebSocketClient

JettyWebSocketClient:
  WebSocket URI: wss://stream.data.alpaca.markets/v2/iex
  Headers:
    - APCA-API-KEY-ID: [your-key]
    - APCA-API-SECRET-KEY: [your-secret]
```

**Subscribe Message (send on connect):**
```json
{
  "action": "subscribe",
  "trades": ["AAPL", "GOOGL", "MSFT"],
  "quotes": ["AAPL", "GOOGL", "MSFT"]
}
```

**Step 2: Parse Trade Data**

**EvaluateJsonPath:**
```properties
Destination: flowfile-attribute

symbol: $.S
price: $.p
size: $.s
timestamp: $.t
exchange: $.x
conditions: $.c
```

**Sample Input:**
```json
{
  "T": "t",
  "S": "AAPL",
  "i": 52983525029461,
  "x": "D",
  "p": 175.23,
  "s": 100,
  "c": ["@"],
  "t": "2024-01-15T14:30:05.123456Z",
  "z": "C"
}
```

**Step 3: Filter Significant Trades**

**QueryRecord:**
```sql
-- Property: significant_trades
SELECT 
  *,
  price * size AS trade_value
FROM FLOWFILE
WHERE size >= 100  -- Block trades only
  AND price > 0
  
-- Property: large_trades  
SELECT 
  symbol,
  price,
  size,
  price * size AS value,
  timestamp
FROM FLOWFILE
WHERE (price * size) > 10000  -- $10k+ trades
ORDER BY value DESC
```

**Step 4: Calculate Technical Indicators**

**UpdateAttribute:**
```properties
# Price change from previous
price.change: ${price:toNumber():minus(${previous.price:toNumber()})}

# Percentage change
price.pct.change: ${price.change:divide(${previous.price:toNumber()}):multiply(100)}

# Volume-weighted price
vwap.update: ${allAttributeNames():contains('vwap'):ifElse(
  ${vwap:multiply(0.95):plus(${price:multiply(0.05)})},
  ${price}
)}

# Alert flag
alert.trigger: ${price.pct.change:toNumber():abs():gt(2)}
```

**Step 5: Generate Trading Signals**

**RouteOnAttribute:**
```properties
buy_signal: 
  ${price.pct.change:toNumber():lt(-1.5)}:and(${size:toNumber():gt(500)})

sell_signal:
  ${price.pct.change:toNumber():gt(1.5)}:and(${size:toNumber():gt(500)})

high_volume:
  ${size:toNumber():gt(1000)}

spike:
  ${price.pct.change:toNumber():abs():gt(3)}
```

**Step 6: Distribute Results**

```
RouteOnAttribute
  ↓ buy_signal
PutKafka (topic: trading.buy.signals)
  
  ↓ sell_signal
PutKafka (topic: trading.sell.signals)
  
  ↓ spike
InvokeHTTP (alert webhook)
  
  ↓ unmatched
PutElasticsearch (store all trades)
```

## WebSocket Chat Application

### Server-Side NiFi Flow

**Create a simple chat server:**

```
Flow 1: Receive Messages
ListenWebSocket
  ↓
UpdateAttribute (add metadata)
  message.received: ${now()}
  session.id: ${websocket.session.id}
  ↓
LogAttribute (audit trail)
  ↓
PutFile (archive messages)
  ↓
ExecuteScript (broadcast to all)
  ↓
[Fan out to all sessions]

Flow 2: Send to All Sessions
ExecuteScript
  ↓ (for each session)
PutWebSocket
```

**ExecuteScript (Python) - Broadcast:**
```python
import json
from org.apache.nifi.processor.io import StreamCallback

class BroadcastCallback(StreamCallback):
    def process(self, inputStream, outputStream):
        # Read incoming message
        text = IOUtils.toString(inputStream, StandardCharsets.UTF_8)
        message_data = json.loads(text)
        
        # Get WebSocket service
        ws_service = context.controllerServiceLookup.getControllerService(
            "ChatWebSocketServer"
        )
        
        # Broadcast to all sessions
        for session in ws_service.getActiveSessions():
            try:
                ws_service.sendMessage(
                    session.sessionId,
                    text,
                    "TEXT"
                )
            except Exception as e:
                log.error(f"Failed to send to {session.sessionId}: {e}")
        
        # Write original message to output
        outputStream.write(text.encode('utf-8'))

flowFile = session.get()
if flowFile:
    flowFile = session.write(flowFile, BroadcastCallback())
    session.transfer(flowFile, REL_SUCCESS)
```

### Client-Side (HTML/JavaScript)

```html
<!DOCTYPE html>
<html>
<head>
    <title>NiFi WebSocket Chat</title>
</head>
<body>
    <div id="messages"></div>
    <input type="text" id="messageInput" placeholder="Type message...">
    <button onclick="sendMessage()">Send</button>

    <script>
        const ws = new WebSocket('ws://nifi-server:8080/chat');
        
        ws.onopen = () => {
            console.log('Connected to NiFi chat');
            addMessage('System', 'Connected to server');
        };
        
        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            addMessage(data.user, data.message);
        };
        
        ws.onerror = (error) => {
            console.error('WebSocket error:', error);
            addMessage('System', 'Connection error');
        };
        
        ws.onclose = () => {
            console.log('Disconnected from chat');
            addMessage('System', 'Disconnected from server');
        };
        
        function sendMessage() {
            const input = document.getElementById('messageInput');
            const message = {
                user: 'User' + Math.floor(Math.random() * 1000),
                message: input.value,
                timestamp: new Date().toISOString()
            };
            
            ws.send(JSON.stringify(message));
            input.value = '';
        }
        
        function addMessage(user, text) {
            const messagesDiv = document.getElementById('messages');
            const messageElement = document.createElement('div');
            messageElement.textContent = `${user}: ${text}`;
            messagesDiv.appendChild(messageElement);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }
    </script>
</body>
</html>
```

## Performance and Scaling

### Connection Management

**Monitor Active Sessions:**

```properties
ExecuteScript (scheduled every 1 min):
  - Count active sessions
  - Log session details
  - Alert on connection limits
```

**Script:**
```groovy
def wsService = context.controllerServiceLookup
    .getControllerService("WebSocketServer")

def activeSessions = wsService.getActiveSessions()
def sessionCount = activeSessions.size()

// Create monitoring FlowFile
def flowFile = session.create()
def attributes = [
    'session.count': sessionCount.toString(),
    'session.limit': '1000',
    'memory.used': "${Runtime.runtime.totalMemory() - Runtime.runtime.freeMemory()}"
]

flowFile = session.putAllAttributes(flowFile, attributes)

// Alert if near limit
if (sessionCount > 800) {
    session.transfer(flowFile, REL_SUCCESS)
} else {
    session.remove(flowFile)
}
```

### Message Batching

For high-volume streams, batch messages:

```
ConnectWebSocket
  ↓
MergeContent
  Merge Strategy: Bin-Packing
  Minimum Entries: 100
  Maximum Entries: 1000
  Max Bin Age: 5 sec
  ↓
SplitJson
  ↓
[Process individual records]
```

### Backpressure Handling

```properties
Connection Settings (between processors):
  Back Pressure Object Threshold: 10,000
  Back Pressure Data Size Threshold: 1 GB
  
ConnectWebSocket:
  Max Wait Time: 30 sec
  Penalize on Failure: true
```

**Handle Slow Consumers:**

```
ConnectWebSocket
  ↓
RouteOnAttribute
  queue.size: ${queue.count:toNumber():lt(5000)}
  ↓ true (normal processing)
  [Regular flow]
  
  ↓ false (queue full)
  LogAttribute (alert)
  ↓
PutFile (overflow to disk)
```

## Security

### SSL/TLS Configuration

**For WSS (Secure WebSocket):**

**Step 1: Create SSL Context Service**

```properties
StandardRestrictedSSLContextService:
  Keystore Filename: /path/to/keystore.jks
  Keystore Password: [encrypted]
  Key Password: [encrypted]
  Keystore Type: JKS
  
  Truststore Filename: /path/to/truststore.jks
  Truststore Password: [encrypted]
  Truststore Type: JKS
  
  TLS Protocol: TLS 1.2
```

**Step 2: Configure WebSocket Server**

```properties
JettyWebSocketServer:
  Listen Port: 8443
  SSL Context Service: SSLContextService
```

**Connect via WSS:**
```javascript
const ws = new WebSocket('wss://nifi-server:8443/secure-endpoint');
```

### Authentication

**Token-Based Auth:**

**Client sends token in initial message:**
```json
{
  "type": "auth",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**NiFi validates token:**
```
ListenWebSocket
  ↓
RouteOnContent (check for auth message)
  ↓ auth
ExecuteScript (validate JWT token)
  ↓ valid
UpdateAttribute (mark session as authenticated)
  session.authenticated: true
  user.id: ${jwt.sub}
  
  ↓ invalid
PutWebSocket (send error and close)
  message: {"error": "Invalid token"}
```

**Authorization Check:**
```
ListenWebSocket
  ↓
RouteOnAttribute
  ${session.authenticated:equals('true')}
  ↓ true (authenticated)
  [Process message]
  
  ↓ false (not authenticated)
PutWebSocket (reject)
  message: {"error": "Not authenticated"}
```

## Error Handling

### Connection Failures

```
ConnectWebSocket
  ↓ failure
LogAttribute (log error details)
  ↓
UpdateAttribute
  retry.count: ${retry.count:plus(1)}
  retry.delay: ${retry.count:multiply(5)}
  ↓
RouteOnAttribute
  ${retry.count:toNumber():lt(5)}
  ↓ true (retry)
Wait (${retry.delay} seconds)
  ↓
[Loop back to ConnectWebSocket]
  
  ↓ false (max retries exceeded)
PutFile (dead letter queue)
```

### Message Validation

```
ListenWebSocket
  ↓
ValidateRecord
  ↓ valid
[Normal processing]
  
  ↓ invalid
UpdateAttribute
  error.type: schema_validation_failed
  error.timestamp: ${now()}
  ↓
PutWebSocket (send error back to client)
  message: {
    "error": "Invalid message format",
    "details": "${validation.error}"
  }
  ↓
LogAttribute
  ↓
PutFile (error archive)
```

### Session Timeout

```properties
JettyWebSocketClient:
  Session Timeout: 1 hour
  Keep-Alive Interval: 30 sec
```

**Implement Ping/Pong:**

```
GenerateFlowFile (every 30 sec)
  ↓
UpdateAttribute
  message.type: ping
  message.timestamp: ${now()}
  ↓
PutWebSocket
  message: {"type":"ping"}
```

**Client responds with pong:**
```javascript
ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'ping') {
        ws.send(JSON.stringify({type: 'pong'}));
    }
};
```

## Monitoring and Debugging

### Logging Strategy

```
ListenWebSocket
  ↓
LogAttribute
  Log Level: DEBUG
  Attributes to Log: websocket.*
  Log Payload: true
  ↓
UpdateAttribute (add tracking)
  message.id: ${UUID()}
  received.at: ${now()}
  session.info: ${websocket.session.id}
```

### Metrics Collection

**Track WebSocket metrics:**

```
ExecuteScript (scheduled every 5 min)
  - Count active connections
  - Calculate message rate
  - Monitor memory usage
  - Detect anomalies
  ↓
ConvertRecord
  (format as metrics)
  ↓
PutElasticsearch
  index: nifi-websocket-metrics
```

**Groovy Script:**
```groovy
import groovy.json.JsonOutput

def wsService = context.controllerServiceLookup
    .getControllerService("WebSocketServer")

def metrics = [
    timestamp: System.currentTimeMillis(),
    active_sessions: wsService.getActiveSessions().size(),
    processor_name: context.name,
    nifi_instance: context.getProperty("nifi.cluster.node.address").value
]

def flowFile = session.create()
flowFile = session.write(flowFile) { outputStream ->
    outputStream.write(JsonOutput.toJson(metrics).bytes)
}

session.transfer(flowFile, REL_SUCCESS)
```

### Testing WebSocket Flows

**1. Use Online WebSocket Tester:**
- https://www.websocket.org/echo.html
- https://www.piesocket.com/websocket-tester

**2. Command-Line Testing (wscat):**

```bash
# Install
npm install -g wscat

# Connect to NiFi WebSocket server
wscat -c ws://localhost:8080

# Send message
> {"sensor_id": "test-001", "value": 25.5}

# Receive response
< {"status": "received", "message_id": "12345"}
```

**3. Python Test Client:**

```python
import asyncio
import websockets
import json

async def test_nifi_websocket():
    uri = "ws://localhost:8080"
    
    async with websockets.connect(uri) as websocket:
        # Send test message
        test_data = {
            "test": True,
            "timestamp": "2024-01-15T10:00:00Z",
            "value": 42
        }
        
        await websocket.send(json.dumps(test_data))
        print(f"Sent: {test_data}")
        
        # Wait for response
        response = await websocket.recv()
        print(f"Received: {response}")

asyncio.run(test_nifi_websocket())
```

## Best Practices

### 1. Design for Failure

- Always handle connection failures
- Implement retry logic with exponential backoff
- Use dead letter queues for undeliverable messages
- Monitor connection health

### 2. Resource Management

- Set appropriate session timeouts
- Limit maximum concurrent connections
- Implement message size limits
- Monitor memory usage

### 3. Message Protocol

- Define clear message schemas
- Version your message format
- Include message IDs for tracking
- Add timestamps to all messages

Example message structure:
```json
{
  "version": "1.0",
  "message_id": "uuid-here",
  "timestamp": "2024-01-15T10:00:00Z",
  "type": "sensor_data",
  "payload": {
    "sensor_id": "temp-001",
    "value": 23.5
  }
}
```

### 4. Security First

- Always use WSS in production
- Implement authentication
- Validate all incoming messages
- Rate limit connections per IP
- Log security events

### 5. Performance Optimization

- Batch small messages when possible
- Use binary frames for large data
- Compress messages if needed
- Monitor and tune thread pools

### 6. Documentation

- Document WebSocket endpoints
- Provide client examples
- Define message formats
- Include error codes

## Common Issues and Solutions

### Issue 1: Connection Drops

**Symptom:** WebSocket connections frequently disconnect

**Solutions:**
```properties
# Increase timeouts
Session Timeout: 3600 sec (1 hour)

# Enable keep-alive
Keep-Alive Interval: 30 sec

# Handle reconnection on client
ws.onclose = () => {
    setTimeout(() => {
        reconnect();
    }, 5000);
};
```

### Issue 2: High Memory Usage

**Symptom:** NiFi memory grows with active connections

**Solutions:**
- Reduce `Max Text Message Size` in server config
- Implement message batching before processing
- Use MergeContent to combine small messages
- Increase heap size: `-Xmx8g`

### Issue 3: Message Loss

**Symptom:** Messages not reaching destination

**Solutions:**
```
# Add acknowledgment flow
ListenWebSocket
  ↓
LogAttribute (capture message)
  ↓
PutWebSocket (send ACK back)
  message: {"ack": "${message.id}"}
  ↓
[Continue processing]
```

### Issue 4: Slow Processing

**Symptom:** Messages queue up faster than processing

**Solutions:**
- Increase concurrent tasks on processors
- Add load balancing with multiple NiFi nodes
- Implement backpressure on client side
- Use Kafka as intermediate buffer

## Summary

WebSocket integration in NiFi enables:
- ✅ Real-time bidirectional communication
- ✅ Low-latency data streaming
- ✅ IoT device integration
- ✅ Live dashboard feeds
- ✅ Event-driven architectures

Key Takeaways:
1. Use WSS (secure WebSocket) in production
2. Implement proper error handling and retries
3. Monitor connection health and resource usage
4. Define clear message protocols
5. Test thoroughly with realistic load

## Next Steps

- Learn [Kafka Integration](11_kafka_integration.md)
- Explore [REST API](14_rest_api.md)
- Try [Database Connections](09_database_connections.md)
- Implement [Wait/Notify Patterns](10_wait_notify.md)
```

## 08_xml_processors.md

```markdown
# XML Processors in Apache NiFi

## Overview

XML (eXtensible Markup Language) is a widely used format for data exchange, especially in enterprise systems, SOAP APIs, and legacy applications. NiFi provides robust processors for parsing, transforming, validating, and generating XML data.

## XML Processors in NiFi

### Core XML Processors

| Processor | Purpose |
|-----------|---------|
| **EvaluateXPath** | Extract values using XPath expressions |
| **EvaluateXQuery** | Transform XML using XQuery |
| **TransformXml** | Apply XSLT transformations |
| **SplitXml** | Split XML into smaller chunks |
| **ValidateXml** | Validate against XSD schema |
| **ConvertRecord** | Convert XML to/from other formats |
| **QueryRecord** | Query XML using SQL |

## Understanding XML Structure

### Basic XML Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
    <book id="bk101">
        <author>Gambardella, Matthew</author>
        <title>XML Developer's Guide</title>
        <genre>Computer</genre>
        <price>44.95</price>
        <publish_date>2000-10-01</publish_date>
        <description>An in-depth look at creating applications with XML.</description>
    </book>
    <book id="bk102">
        <author>Ralls, Kim</author>
        <title>Midnight Rain</title>
        <genre>Fantasy</genre>
        <price>5.95</price>
        <publish_date>2000-12-16</publish_date>
        <description>A former architect battles corporate zombies.</description>
    </book>
</catalog>
```

### XML Namespaces

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope 
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <soap:Header>
        <authentication>
            <username>user123</username>
            <token>abc-def-ghi</token>
        </authentication>
    </soap:Header>
    <soap:Body>
        <getCustomer xmlns="http://example.com/customers">
            <customerId>12345</customerId>
        </getCustomer>
    </soap:Body>
</soap:Envelope>
```

## EvaluateXPath Processor

### Overview

Extracts data from XML using XPath expressions and stores results in FlowFile attributes.

### Configuration

```properties
Destination: flowfile-attribute
Return Type: string (or nodeset, boolean, number)
```

### XPath Examples

**Basic Path Selection:**

```xml
<!-- Input XML -->
<person>
    <name>John Doe</name>
    <age>30</age>
    <email>john@example.com</email>
</person>
```

```properties
EvaluateXPath Properties:
  name: /person/name
  age: /person/age
  email: /person/email
```

**Result:**
```
Attributes:
  name = "John Doe"
  age = "30"
  email = "john@example.com"
```

**Attribute Selection:**

```xml
<book id="123" category="fiction">
    <title>Great Book</title>
    <price currency="USD">29.99</price>
</book>
```

```properties
EvaluateXPath Properties:
  book.id: /book/@id
  book.category: /book/@category
  price.value: /book/price
  price.currency: /book/price/@currency
```

**Nested Elements:**

```xml
<order>
    <customer>
        <name>Jane Smith</name>
        <address>
            <street>123 Main St</street>
            <city>New York</city>
            <zip>10001</zip>
        </address>
    </customer>
    <items>
        <item>
            <product>Widget</product>
            <quantity>5</quantity>
        </item>
    </items>
</order>
```

```properties
EvaluateXPath Properties:
  customer.name: /order/customer/name
  customer.city: /order/customer/address/city
  customer.zip: /order/customer/address/zip
  first.product: /order/items/item[1]/product
  first.quantity: /order/items/item[1]/quantity
```

**XPath Functions:**

```properties
# Count elements
item.count: count(/order/items/item)

# String operations
title.upper: upper-case(/book/title)
title.lower: lower-case(/book/title)
title.length: string-length(/book/title)

# Numeric operations
total.price: sum(/order/items/item/price)
avg.price: avg(/order/items/item/price)
max.price: max(/order/items/item/price)

# Conditional selection
expensive.items: /order/items/item[price > 100]/product
available.items: /order/items/item[@status='available']/product
```

**With Namespaces:**

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <ns:getCustomerResponse xmlns:ns="http://example.com/customer">
            <ns:customer>
                <ns:id>12345</ns:id>
                <ns:name>John Doe</ns:name>
            </ns:customer>
        </ns:getCustomerResponse>
    </soap:Body>
</soap:Envelope>
```

```properties
EvaluateXPath Properties:
  # Define namespaces first
  soap: http://schemas.xmlsoap.org/soap/envelope/
  ns: http://example.com/customer
  
  # Then use them in XPath
  customer.id: //ns:customer/ns:id
  customer.name: //ns:customer/ns:name
```

### Return Types

**String (default):**
```properties
title: /book/title
Result: "XML Developer's Guide"
```

**Number:**
```properties
Return Type: number
price: /book/price
Result: 44.95 (as number)
```

**Boolean:**
```properties
Return Type: boolean
has.description: boolean(/book/description)
Result: true or false
```

**Nodeset:**
```properties
Return Type: nodeset
all.authors: /catalog/book/author
Result: XML fragment containing all author elements
```

## SplitXml Processor

### Overview

Splits large XML files into smaller chunks based on a specified element depth.

### Configuration

```properties
Split Depth: 2
```

### Example: Split Books

**Input:**
```xml
<catalog>
    <book id="bk101">
        <title>Book 1</title>
        <author>Author 1</author>
    </book>
    <book id="bk102">
        <title>Book 2</title>
        <author>Author 2</author>
    </book>
    <book id="bk103">
        <title>Book 3</title>
        <author>Author 3</author>
    </book>
</catalog>
```

**With Split Depth = 2:**

Output FlowFile 1:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<book id="bk101">
    <title>Book 1</title>
    <author>Author 1</author>
</book>
```

Output FlowFile 2:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<book id="bk102">
    <title>Book 2</title>
    <author>Author 2</author>
</book>
```

Output FlowFile 3:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<book id="bk103">
    <title>Book 3</title>
    <author>Author 3</author>
</book>
```

### Understanding Split Depth

```
Depth 0: (root)
Depth 1: <catalog>
Depth 2:     <book>
Depth 3:         <title>, <author>, etc.
```

```properties
Split Depth: 1
  Result: Entire <catalog> (no split)

Split Depth: 2
  Result: Individual <book> elements

Split Depth: 3
  Result: Individual <title>, <author> elements
```

### Advanced Splitting

**Complex XML:**
```xml
<company>
    <departments>
        <department name="Sales">
            <employee id="001">
                <name>John</name>
                <salary>50000</salary>
            </employee>
            <employee id="002">
                <name>Jane</name>
                <salary>60000</salary>
            </employee>
        </department>
        <department name="IT">
            <employee id="003">
                <name>Bob</name>
                <salary>70000</salary>
            </employee>
        </department>
    </departments>
</company>
```

```properties
Split Depth: 3
  Splits into department elements

Split Depth: 4
  Splits into individual employee elements
```

## TransformXml (XSLT) Processor

### Overview

Applies XSLT transformations to convert XML structure and content.

### Configuration

```properties
XSLT File Name: /path/to/transform.xsl
  OR
XSLT Content: [paste XSLT directly]
```

### XSLT Example: XML to HTML

**Input XML:**
```xml
<catalog>
    <book>
        <title>XML Guide</title>
        <author>John Doe</author>
        <price>29.99</price>
    </book>
    <book>
        <title>XSLT Cookbook</title>
        <author>Jane Smith</author>
        <price>34.99</price>
    </book>
</catalog>
```

**XSLT Transformation:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:output method="html" indent="yes"/>
    
    <xsl:template match="/catalog">
        <html>
            <head>
                <title>Book Catalog</title>
                <style>
                    table { border-collapse: collapse; width: 100%; }
                    th, td { border: 1px solid black; padding: 8px; }
                    th { background-color: #4CAF50; color: white; }
                </style>
            </head>
            <body>
                <h1>Book Catalog</h1>
                <table>
                    <tr>
                        <th>Title</th>
                        <th>Author</th>
                        <th>Price</th>
                    </tr>
                    <xsl:apply-templates select="book"/>
                </table>
            </body>
        </html>
    </xsl:template>
    
    <xsl:template match="book">
        <tr>
            <td><xsl:value-of select="title"/></td>
            <td><xsl:value-of select="author"/></td>
            <td>$<xsl:value-of select="price"/></td>
        </tr>
    </xsl:template>
</xsl:stylesheet>
```

**Output HTML:**
```html
<html>
    <head>
        <title>Book Catalog</title>
        <style>...</style>
    </head>
    <body>
        <h1>Book Catalog</h1>
        <table>
            <tr>
                <th>Title</th>
                <th>Author</th>
                <th>Price</th>
            </tr>
            <tr>
                <td>XML Guide</td>
                <td>John Doe</td>
                <td>$29.99</td>
            </tr>
            <tr>
                <td>XSLT Cookbook</td>
                <td>Jane Smith</td>
                <td>$34.99</td>
            </tr>
        </table>
    </body>
</html>
```

### XSLT Example: XML Restructuring

**Input:**
```xml
<orders>
    <order>
        <id>001</id>
        <customer>John Doe</customer>
        <item>Widget</item>
        <quantity>5</quantity>
        <price>10.00</price>
    </order>
</orders>
```

**XSLT to Reorganize:**
```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/orders">
        <transformed_orders>
            <xsl:apply-templates select="order"/>
        </transformed_orders>
    </xsl:template>
    
    <xsl:template match="order">
        <order_record>
            <order_info>
                <order_id><xsl:value-of select="id"/></order_id>
                <customer_name><xsl:value-of select="customer"/></customer_name>
            </order_info>
            <line_item>
                <product><xsl:value-of select="item"/></product>
                <qty><xsl:value-of select="quantity"/></qty>
                <unit_price><xsl:value-of select="price"/></unit_price>
                <total>
                    <xsl:value-of select="quantity * price"/>
                </total>
            </line_item>
        </order_record>
    </xsl:template>
</xsl:stylesheet>
```

**Output:**
```xml
<transformed_orders>
    <order_record>
        <order_info>
            <order_id>001</order_id>
            <customer_name>John Doe</customer_name>
        </order_info>
        <line_item>
            <product>Widget</product>
            <qty>5</qty>
            <unit_price>10.00</unit_price>
            <total>50.00</total>
        </line_item>
    </order_record>
</transformed_orders>
```

### XSLT with Filtering

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/catalog">
        <filtered_catalog>
            <!-- Only books with price > 30 -->
            <xsl:apply-templates select="book[price &gt; 30]"/>
        </filtered_catalog>
    </xsl:template>
    
    <xsl:template match="book">
        <book>
            <xsl:copy-of select="title"/>
            <xsl:copy-of select="price"/>
            <!-- Add discount -->
            <discounted_price>
                <xsl:value-of select="price * 0.9"/>
            </discounted_price>
        </book>
    </xsl:template>
</xsl:stylesheet>
```

## ValidateXml Processor

### Overview

Validates XML against an XSD (XML Schema Definition) to ensure data integrity.

### Configuration

```properties
Schema File: /path/to/schema.xsd
  OR
Schema Text: [paste XSD directly]
```

### XSD Schema Example

**Schema (customer.xsd):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="customer">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="id" type="xs:integer"/>
                <xs:element name="name" type="xs:string"/>
                <xs:element name="email" type="emailType"/>
                <xs:element name="age" type="ageType"/>
                <xs:element name="status" type="statusType"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    
    <!-- Custom email type with pattern -->
    <xs:simpleType name="emailType">
        <xs:restriction base="xs:string">
            <xs:pattern value="[^@]+@[^@]+\.[^@]+"/>
        </xs:restriction>
    </xs:simpleType>
    
    <!-- Age must be between 0 and 120 -->
    <xs:simpleType name="ageType">
        <xs:restriction base="xs:integer">
            <xs:minInclusive value="0"/>
            <xs:maxInclusive value="120"/>
        </xs:restriction>
    </xs:simpleType>
    
    <!-- Status must be one of: active, inactive, suspended -->
    <xs:simpleType name="statusType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="active"/>
            <xs:enumeration value="inactive"/>
            <xs:enumeration value="suspended"/>
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```

**Valid XML:**
```xml
<customer>
    <id>12345</id>
    <name>John Doe</name>
    <email>john@example.com</email>
    <age>30</age>
    <status>active</status>
</customer>
```

**Invalid XML (will fail validation):**
```xml
<customer>
    <id>12345</id>
    <name>John Doe</name>
    <email>invalid-email</email>  <!-- Bad format -->
    <age>150</age>                <!-- Out of range -->
    <status>pending</status>      <!-- Invalid value -->
</customer>
```

### Validation Flow

```
GetFile
  ↓
ValidateXml
  ↓ valid
PutDatabaseRecord
  
  ↓ invalid
LogAttribute (log validation errors)
  ↓
UpdateAttribute
  validation.error: ${ValidateXml.error}
  ↓
PutFile (quarantine directory)
```

## ConvertRecord with XML

### Overview

Convert XML to/from other formats (JSON, CSV, Avro) using record-based processing.

### XML to JSON

**Configuration:**
```properties
Record Reader: XMLReader
Record Writer: JsonRecordSetWriter
```

**XMLReader Configuration:**
```properties
Schema Access Strategy: Infer Schema
Expect Records as Array: false
```

**Input XML:**
```xml
<records>
    <record>
        <id>1</id>
        <name>Alice</name>
        <email>alice@example.com</email>
    </record>
    <record>
        <id>2</id>
        <name>Bob</name>
        <email>bob@example.com</email>
    </record>
</records>
```

**Output JSON:**
```json
[
  {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "id": 2,
    "name": "Bob",
    "email": "bob@example.com"
  }
]
```

### XML to CSV

**Configuration:**
```properties
Record Reader: XMLReader
Record Writer: CSVRecordSetWriter
```

**CSVRecordSetWriter:**
```properties
Include Header Line: true
```

**Output:**
```csv
id,name,email
1,Alice,alice@example.com
2,Bob,bob@example.com
```

### JSON to XML

**Configuration:**
```properties
Record Reader: JsonTreeReader
Record Writer: XMLRecordSetWriter
```

**XMLRecordSetWriter:**
```properties
Root Tag Name: records
Record Tag Name: record
```

**Input JSON:**
```json
[
  {"id": 1, "name": "Alice"},
  {"id": 2, "name": "Bob"}
]
```

**Output XML:**
```xml
<records>
    <record>
        <id>1</id>
        <name>Alice</name>
    </record>
    <record>
        <id>2</id>
        <name>Bob</name>
    </record>
</records>
```

## QueryRecord with XML

### Overview

Use SQL to query and transform XML data.

**Configuration:**
```properties
Record Reader: XMLReader
Record Writer: XMLRecordSetWriter (or JsonRecordSetWriter)
```

**Input XML:**
```xml
<employees>
    <employee>
        <id>1</id>
        <name>Alice</name>
        <department>IT</department>
        <salary>75000</salary>
    </employee>
    <employee>
        <id>2</id>
        <name>Bob</name>
        <department>Sales</department>
        <salary>65000</salary>
    </employee>
    <employee>
        <id>3</id>
        <name>Charlie</name>
        <department>IT</department>
        <salary>80000</salary>
    </employee>
</employees>
```

**SQL Queries:**
```sql
-- Property: it_employees
SELECT * FROM FLOWFILE
WHERE department = 'IT'

-- Property: high_earners
SELECT * FROM FLOWFILE
WHERE salary > 70000

-- Property: dept_summary
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary
FROM FLOWFILE
GROUP BY department
```

## SOAP Web Service Processing

### Complete SOAP Request/Response Flow

**Scenario:** Call SOAP web service and process response

```
Flow:
GenerateFlowFile (or trigger)
  ↓
ReplaceText (create SOAP request)
  ↓
InvokeHTTP (call SOAP service)
  ↓
EvaluateXPath (extract response data)
  ↓
RouteOnAttribute (check success)
  ↓ success / failure
[Process accordingly]
```

**Step 1: Create SOAP Request**

**ReplaceText:**
```properties
Replacement Strategy: Always Replace
Replacement Value:
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope 
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:cust="http://example.com/customer">
    <soap:Header>
        <authentication>
            <username>${username}</username>
            <password>${password}</password>
        </authentication>
    </soap:Header>
    <soap:Body>
        <cust:getCustomer>
            <cust:customerId>${customer.id}</cust:customerId>
        </cust:getCustomer>
    </soap:Body>
</soap:Envelope>
```

**Step 2: Call SOAP Service**

**InvokeHTTP:**
```properties
HTTP Method: POST
Remote URL: http://example.com/CustomerService
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://example.com/getCustomer"
```

**Step 3: Parse SOAP Response**

**Sample Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <getCustomerResponse xmlns="http://example.com/customer">
            <customer>
                <id>12345</id>
                <name>John Doe</name>
                <email>john@example.com</email>
                <status>active</status>
                <balance>1250.50</balance>
            </customer>
        </getCustomerResponse>
    </soap:Body>
</soap:Envelope>
```

**EvaluateXPath:**
```properties
# Define namespaces
soap: http://schemas.xmlsoap.org/soap/envelope/
cust: http://example.com/customer

# Extract values
customer.id: //cust:customer/cust:id
customer.name: //cust:customer/cust:name
customer.email: //cust:customer/cust:email
customer.status: //cust:customer/cust:status
customer.balance: //cust:customer/cust:balance
```

**Step 4: Handle SOAP Faults**

**SOAP Fault Response:**
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <soap:Fault>
            <faultcode>soap:Client</faultcode>
            <faultstring>Customer not found</faultstring>
            <detail>
                <errorCode>404</errorCode>
                <message>Customer ID 12345 does not exist</message>
            </detail>
        </soap:Fault>
    </soap:Body>
</soap:Envelope>
```

**Check for Fault:**
```properties
EvaluateXPath:
  is.fault: boolean(//soap:Fault)
  fault.code: //soap:Fault/faultcode
  fault.message: //soap:Fault/faultstring
  error.code: //soap:Fault/detail/errorCode
```

**RouteOnAttribute:**
```properties
success: ${is.fault:equals('false')}
fault: ${is.fault:equals('true')}
```

## XML Processing Patterns

### Pattern 1: Large XML File Processing

**Challenge:** Process 1GB XML file with millions of records

```
GetFile (large XML)
  ↓
SplitXml (split into records)
  ↓
MergeContent (batch 100 records)
  ↓
ConvertRecord (XML → JSON)
  ↓
PutElasticsearch
```

### Pattern 2: XML Validation Pipeline

```
GetFile
  ↓
ValidateXml
  ↓ valid
EvaluateXPath (extract data)
  ↓
ConvertRecord (XML → JSON)
  ↓
PutDatabaseRecord
  
  ↓ invalid (from ValidateXml)
LogAttribute
  ↓
UpdateAttribute (add error details)
  ↓
PutFile (error directory)
  ↓
InvokeHTTP (send alert)
```

### Pattern 3: XML Transformation Chain

```
GetFile (source XML)
  ↓
TransformXml (XSLT #1: normalize structure)
  ↓
EvaluateXPath (extract metadata)
  ↓
RouteOnAttribute (by document type)
  ↓ invoice
TransformXml (XSLT #2: invoice-specific)
  ↓
ValidateXml (invoice.xsd)
  ↓
ConvertRecord (XML → JSON)
  ↓
PutKafka (invoice topic)
```

### Pattern 4: Multi-Source XML Aggregation

```
Flow 1:
GetFile (source A - XML)
  ↓
UpdateAttribute (source: A)
  ↓
[Merge Point]

Flow 2:
InvokeHTTP (source B - SOAP)
  ↓
UpdateAttribute (source: B)
  ↓
[Merge Point]

[Merge Point]:
MergeContent
  ↓
ConvertRecord (all XML → JSON)
  ↓
QueryRecord (aggregate)
  ↓
PutElasticsearch
```

## Complex Example: Order Processing System

### Scenario

Process XML orders from multiple sources:
- FTP uploads
- SOAP web service calls
- HTTP POST submissions

Validate, transform, enrich, and route to appropriate systems.

### Complete Flow

```
[Multiple Sources]
  ↓
MergeContent (standardize format)
  ↓
ValidateXml (order.xsd)
  ↓ valid
EvaluateXPath (extract order details)
  order.id: //order/id
  customer.id: //order/customerId
  order.total: //order/total
  order.priority: //order/priority
  ↓
InvokeHTTP (enrich with customer data)
  URL: http://customer-api/customers/${customer.id}
  ↓
JoltTransformJSON (merge customer data)
  ↓
QueryRecord (calculate totals, validate)
  ↓ high_value (total > 1000)
TransformXml (XSLT: add VIP processing)
  ↓
ConvertRecord (XML → JSON)
  ↓
PutKafka (priority-orders topic)
  
  ↓ standard (from QueryRecord)
ConvertRecord (XML → JSON)
  ↓
PutKafka (standard-orders topic)
  
  ↓ invalid (from ValidateXml)
UpdateAttribute
  error.timestamp: ${now()}
  error.details: ${ValidateXml.error}
  ↓
PutFile (quarantine/)
  ↓
InvokeHTTP (alert ops team)
```

### XSD Schema (order.xsd)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="order">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="id" type="xs:string"/>
                <xs:element name="customerId" type="xs:integer"/>
                <xs:element name="orderDate" type="xs:dateTime"/>
                <xs:element name="priority" type="priorityType"/>
                <xs:element name="items" type="itemsType"/>
                <xs:element name="total" type="xs:decimal"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    
    <xs:simpleType name="priorityType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="low"/>
            <xs:enumeration value="normal"/>
            <xs:enumeration value="high"/>
            <xs:enumeration value="urgent"/>
        </xs:restriction>
    </xs:simpleType>
    
    <xs:complexType name="itemsType">
        <xs:sequence>
            <xs:element name="item" maxOccurs="unbounded">
                <xs:complexType>
                    <xs:sequence>
                        <xs:element name="sku" type="xs:string"/>
                        <xs:element name="quantity" type="xs:positiveInteger"/>
                        <xs:element name="price" type="xs:decimal"/>
                    </xs:sequence>
                </xs:complexType>
            </xs:element>
        </xs:sequence>
    </xs:complexType>
</xs:schema>
```

### Sample Order XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<order>
    <id>ORD-2024-001</id>
    <customerId>12345</customerId>
    <orderDate>2024-01-15T10:30:00Z</orderDate>
    <priority>high</priority>
    <items>
        <item>
            <sku>WIDGET-001</sku>
            <quantity>5</quantity>
            <price>29.99</price>
        </item>
        <item>
            <sku>GADGET-002</sku>
            <quantity>2</quantity>
            <price>149.99</price>
        </item>
    </items>
    <total>449.93</total>
</order>
```

## Performance Optimization

### 1. Schema Caching

**Use Avro Schema Registry for repeated conversions:**

```properties
XMLReader:
  Schema Access Strategy: Use 'Schema Name' Property
  Schema Registry: AvroSchemaRegistry
  Schema Name: order-schema
```

### 2. Parallel Processing

```
SplitXml
  ↓
UpdateAttribute (add partition key)
  partition: ${UUID():substring(0,1)}
  ↓
RouteOnAttribute (partition by first char)
  ↓ 0-7 / 8-f
[Separate processing lanes]
```

### 3. Batch Processing

```
GetFile
  ↓
SplitXml (individual records)
  ↓
MergeContent
  Minimum Entries: 1000
  Maximum Entries: 5000
  Max Bin Age: 30 sec
  ↓
ConvertRecord
```

### 4. Memory Management

For large XML files:

```properties
nifi.nar.unpack.uber.jar: false
nifi.content.claim.max.appendable.size: 1 MB
nifi.content.claim.max.flow.files: 100
```

Increase heap for XML processing:
```
java.arg.2=-Xms4g
java.arg.3=-Xmx8g
```

## Error Handling Best Practices

### 1. Comprehensive Validation

```
GetFile
  ↓
ValidateXml (schema validation)
  ↓ valid
EvaluateXPath (business rule validation)
  has.required.fields: boolean(//order/id and //order/customerId)
  ↓
RouteOnAttribute
  ${has.required.fields:equals('true')}
  ↓ true
[Continue processing]
  
  ↓ false
LogAttribute
  ↓
PutFile (missing-fields/)
```

### 2. Graceful Degradation

```
EvaluateXPath
  ↓ failure (malformed XML)
LogAttribute (log error)
  ↓
ReplaceText (extract what's possible with regex)
  ↓
[Partial processing or quarantine]
```

### 3. Detailed Error Logging

```properties
LogAttribute Configuration:
  Log Level: ERROR
  Attributes to Log: 
    - ValidateXml.error
    - filename
    - path
    - absolute.path
  Attributes to Log Regex: .*error.*
  Log Payload: true (for debugging)
```

### 4. Dead Letter Queue

```
[Any Processor]
  ↓ failure
UpdateAttribute
  dlq.reason: XML_PROCESSING_FAILED
  dlq.processor: ${processor.name}
  dlq.timestamp: ${now()}
  dlq.original.filename: ${filename}
  ↓
PutFile
  Directory: /data/dlq/${dlq.reason}
  ↓
InvokeHTTP (notify monitoring system)
```

## Testing XML Flows

### 1. Sample Data Generator

```
GenerateFlowFile
  Custom Text:
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testOrder>
    <id>TEST-${UUID()}</id>
    <timestamp>${now()}</timestamp>
    <testData>true</testData>
</testOrder>
```

### 2. Validation Testing

```
GenerateFlowFile (valid XML)
  ↓
ValidateXml
  ↓ valid
LogAttribute (success counter)

GenerateFlowFile (invalid XML)
  ↓
ValidateXml
  ↓ invalid
LogAttribute (error counter)
```

### 3. Transformation Testing

```
GetFile (test data/)
  ↓
TransformXml
  ↓
PutFile (output/)
  ↓
[Manually verify output]
```

## Common Issues and Solutions

### Issue 1: Namespace Problems

**Problem:** XPath not finding elements with namespaces

**Solution:**
```properties
EvaluateXPath:
  # Define ALL namespaces used in XML
  ns1: http://example.com/namespace1
  ns2: http://example.com/namespace2
  
  # Use namespaces in XPath
  value: //ns1:root/ns2:element
```

### Issue 2: Large File Memory Issues

**Problem:** OutOfMemoryError with large XML files

**Solution:**
```
# Split before processing
GetFile
  ↓
SplitXml (split into chunks)
  ↓
[Process chunks individually]

# Or use streaming
GetFile
  ↓
ExecuteStreamCommand
  Command: xmlstarlet sel -t -c "//record"
  ↓
SplitText
```

### Issue 3: Invalid Characters

**Problem:** XML contains invalid UTF-8 characters

**Solution:**
```
GetFile
  ↓
ExecuteStreamCommand
  Command: iconv -f UTF-8 -t UTF-8 -c
  ↓
ValidateXml
```

Or use ReplaceText:
```properties
Search Value: [^\x09\x0A\x0D\x20-\xD7FF\xE000-\xFFFD\x10000-x10FFFF]
Replacement Value: (empty)
Replacement Strategy: Regex Replace
```

### Issue 4: XSLT Transformation Errors

**Problem:** XSLT not producing expected output

**Solution:**
1. Test XSLT externally first
2. Add debug output to XSLT:
```xml
<xsl:message>Processing: <xsl:value-of select="name()"/></xsl:message>
```
3. Check XSLT version compatibility
4. Verify namespace declarations

## Best Practices

### 1. Always Validate

```
[Source]
  ↓
ValidateXml (first step)
  ↓ valid
[Continue processing]
  
  ↓ invalid
[Error handling]
```

### 2. Use Namespaces Consistently

Define namespaces in one place:
```properties
UpdateAttribute (at flow start):
  ns.soap: http://schemas.xmlsoap.org/soap/envelope/
  ns.app: http://example.com/application
  
Then reference: ${ns.soap}, ${ns.app}
```

### 3. Document XPath Expressions

```properties
# Extract customer email
# Example: <customer><email>test@example.com</email></customer>
customer.email: //customer/email

# Get total order count
# Returns number of <order> elements
order.count: count(//order)
```

### 4. Handle Optional Elements

```properties
# Use default for missing elements
phone: concat(//customer/phone, '')

# Or check existence first
has.phone: boolean(//customer/phone)
phone: //customer/phone
```

### 5. Version Your Schemas

```
schemas/
  order-v1.xsd
  order-v2.xsd
  customer-v1.xsd
```

Include version in processing:
```properties
UpdateAttribute:
  schema.version: v2
  
ValidateXml:
  Schema File: /schemas/${schema.type}-${schema.version}.xsd
```

## Summary

XML processing in NiFi provides:
- ✅ XPath extraction with EvaluateXPath
- ✅ XSLT transformations with TransformXml
- ✅ Schema validation with ValidateXml
- ✅ Format conversion with ConvertRecord
- ✅ SQL queries with QueryRecord
- ✅ Large file handling with SplitXml

Key Takeaways:
1. Always validate XML before processing
2. Use appropriate processor for the task
3. Handle namespaces correctly in XPath
4. Consider memory when processing large files
5. Implement comprehensive error handling
6. Test transformations thoroughly

```
## 09_database_connections.md

```markdown
# Database Connections in Apache NiFi

## Overview

Apache NiFi provides robust database connectivity through JDBC-based processors and controller services. This guide covers connecting to various databases, executing queries, performing CRUD operations, and implementing database integration patterns.

## Database Processors

### Core Database Processors

| Processor | Purpose |
|-----------|---------|
| **ExecuteSQL** | Execute SELECT queries, return results as FlowFiles |
| **ExecuteSQLRecord** | Execute SELECT queries with record-based output |
| **PutSQL** | Execute INSERT/UPDATE/DELETE statements |
| **PutDatabaseRecord** | Insert records from structured data |
| **QueryDatabaseTable** | Incrementally fetch table data |
| **QueryDatabaseTableRecord** | Record-based incremental table queries |
| **GenerateTableFetch** | Generate queries for partitioned fetching |
| **ListDatabaseTables** | List all tables in a database |

## Controller Services

### DBCPConnectionPool

Central service for managing database connections.

**Configuration:**
```properties
Database Connection URL: jdbc:mysql://localhost:3306/mydb
Database Driver Class Name: com.mysql.jdbc.Driver
Database Driver Location: /path/to/mysql-connector.jar
Database User: username
Password: password
Max Wait Time: 500 millis
Max Total Connections: 8
Validation Query: SELECT 1
```

### Database-Specific Connection Strings

**MySQL/MariaDB:**
```properties
URL: jdbc:mysql://localhost:3306/database_name?useSSL=false&serverTimezone=UTC
Driver: com.mysql.cj.jdbc.Driver
JAR: mysql-connector-java-8.0.x.jar
```

**PostgreSQL:**
```properties
URL: jdbc:postgresql://localhost:5432/database_name
Driver: org.postgresql.Driver
JAR: postgresql-42.x.x.jar
```

**SQL Server:**
```properties
URL: jdbc:sqlserver://localhost:1433;databaseName=mydb;encrypt=true;trustServerCertificate=true
Driver: com.microsoft.sqlserver.jdbc.SQLServerDriver
JAR: mssql-jdbc-x.x.x.jre8.jar
```

**Oracle:**
```properties
URL: jdbc:oracle:thin:@localhost:1521:orcl
Driver: oracle.jdbc.driver.OracleDriver
JAR: ojdbc8.jar
```

**SQLite:**
```properties
URL: jdbc:sqlite:/path/to/database.db
Driver: org.sqlite.JDBC
JAR: sqlite-jdbc-x.x.x.jar
```

**Snowflake:**
```properties
URL: jdbc:snowflake://account.region.snowflakecomputing.com/?warehouse=WH&db=DB&schema=SCHEMA
Driver: net.snowflake.client.jdbc.SnowflakeDriver
JAR: snowflake-jdbc-x.x.x.jar
```

**Databricks:**
```properties
URL: jdbc:databricks://workspace.cloud.databricks.com:443/default;transportMode=http;ssl=1;httpPath=/sql/1.0/endpoints/xxxxx;AuthMech=3;UID=token;PWD=<personal-access-token>
Driver: com.databricks.client.jdbc.Driver
JAR: DatabricksJDBC42-x.x.x.jar
```

## Installing Database Drivers

### Method 1: Manual Installation

**Step 1: Download Driver**
- MySQL: https://dev.mysql.com/downloads/connector/j/
- PostgreSQL: https://jdbc.postgresql.org/download/
- SQL Server: https://docs.microsoft.com/en-us/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server

**Step 2: Place in NiFi**
```bash
# Option A: NiFi lib directory
cp mysql-connector-java-8.0.33.jar /opt/nifi/lib/

# Option B: Custom directory
mkdir -p /opt/nifi/jdbc-drivers
cp *.jar /opt/nifi/jdbc-drivers/
```

**Step 3: Configure DBCPConnectionPool**
```properties
Database Driver Location(s): /opt/nifi/jdbc-drivers/mysql-connector-java-8.0.33.jar
```

### Method 2: Using NiFi Registry

For clustered environments, use NiFi Registry to share driver configurations.

## ExecuteSQL Processor

### Overview

Executes SELECT queries and returns results as FlowFiles in Avro format.

### Configuration

```properties
Database Connection Pooling Service: DBCPConnectionPool
SQL Select Query: SELECT * FROM customers WHERE status = 'active'
Max Rows Per Flow File: 0 (unlimited)
Output Batch Size: 0 (all results in one FlowFile)
```

### Examples

**Example 1: Simple SELECT**

```
ExecuteSQL:
  SQL Select Query: SELECT id, name, email FROM users
  ↓
ConvertAvroToJSON
  ↓
PutFile
```

**SQL Query:**
```sql
SELECT 
    id,
    name,
    email,
    created_at
FROM users
WHERE active = true
ORDER BY created_at DESC
LIMIT 100
```

**Example 2: Parameterized Query**

```
GenerateFlowFile
  ↓
UpdateAttribute
  user.status: active
  min.balance: 1000
  ↓
ExecuteSQL
  SQL: SELECT * FROM customers 
       WHERE status = ? AND balance >= ?
  SQL Arguments: ${user.status}, ${min.balance}
```

**Example 3: Dynamic Query from FlowFile**

```
GetFile (contains SQL query)
  ↓
ExecuteSQL
  SQL Select Query: ${sql.query}  # Read from file content or attribute
```

**Example 4: Join Multiple Tables**

```sql
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email,
    p.product_name,
    oi.quantity,
    oi.price,
    (oi.quantity * oi.price) AS line_total
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '30' DAY
ORDER BY o.order_date DESC
```

## ExecuteSQLRecord Processor

### Overview

Similar to ExecuteSQL but with configurable output format (JSON, CSV, Avro, etc.).

### Configuration

```properties
Database Connection Pooling Service: DBCPConnectionPool
SQL Select Query: SELECT * FROM orders
Record Writer: JsonRecordSetWriter
Include Zero Record FlowFiles: false
```

### Example Flow

```
ExecuteSQLRecord
  Record Writer: JsonRecordSetWriter
  SQL: SELECT * FROM orders WHERE created_date = CURRENT_DATE
  ↓
QueryRecord (filter/transform)
  ↓
PutElasticsearch
```

**Output (JSON):**
```json
[
  {
    "order_id": 1001,
    "customer_id": 5001,
    "total": 299.99,
    "created_date": "2024-01-15"
  },
  {
    "order_id": 1002,
    "customer_id": 5002,
    "total": 450.00,
    "created_date": "2024-01-15"
  }
]
```

## PutSQL Processor

### Overview

Executes INSERT, UPDATE, DELETE, or other DML statements from FlowFile content.

### Configuration

```properties
Database Connection Pooling Service: DBCPConnectionPool
Support Fragmented Transactions: true
Batch Size: 100
```

### SQL Statement Format

FlowFile content must contain SQL statements:

**INSERT Example:**
```sql
INSERT INTO customers (name, email, status) 
VALUES ('John Doe', 'john@example.com', 'active');

INSERT INTO customers (name, email, status) 
VALUES ('Jane Smith', 'jane@example.com', 'active');
```

**UPDATE Example:**
```sql
UPDATE customers 
SET status = 'inactive', updated_at = CURRENT_TIMESTAMP 
WHERE last_login < CURRENT_DATE - INTERVAL '90' DAY;
```

**DELETE Example:**
```sql
DELETE FROM temp_data 
WHERE created_at < CURRENT_DATE - INTERVAL '7' DAY;
```

### Complete Flow Example

```
GetFile (SQL scripts)
  ↓
ReplaceText (replace placeholders)
  Search Value: \$\{date\}
  Replacement: ${now():format('yyyy-MM-dd')}
  ↓
PutSQL
  ↓ success
LogAttribute (log row count)
  ↓
DeleteFile
  
  ↓ failure
LogAttribute (log error)
  ↓
PutFile (error directory)
```

## PutDatabaseRecord Processor

### Overview

Inserts records from structured data (JSON, CSV, Avro) into database tables. Automatically generates INSERT statements.

### Configuration

```properties
Record Reader: JsonTreeReader
Database Connection Pooling Service: DBCPConnectionPool
Statement Type: INSERT
Table Name: customers
Translate Field Names: true
Unmatched Field Behavior: Ignore
Unmatched Column Behavior: Fail
Update Keys: (for UPSERT)
```

### Statement Types

**INSERT:**
```properties
Statement Type: INSERT
Table Name: orders
```

Automatically generates:
```sql
INSERT INTO orders (order_id, customer_id, total, created_date)
VALUES (?, ?, ?, ?)
```

**UPDATE:**
```properties
Statement Type: UPDATE
Table Name: customers
Update Keys: customer_id
```

Generates:
```sql
UPDATE customers 
SET name = ?, email = ?, status = ?
WHERE customer_id = ?
```

**UPSERT (INSERT or UPDATE):**
```properties
Statement Type: UPSERT
Table Name: inventory
Update Keys: product_id
```

**MySQL:**
```sql
INSERT INTO inventory (product_id, quantity, updated_at)
VALUES (?, ?, ?)
ON DUPLICATE KEY UPDATE quantity = VALUES(quantity), updated_at = VALUES(updated_at)
```

**PostgreSQL:**
```sql
INSERT INTO inventory (product_id, quantity, updated_at)
VALUES (?, ?, ?)
ON CONFLICT (product_id) 
DO UPDATE SET quantity = EXCLUDED.quantity, updated_at = EXCLUDED.updated_at
```

**DELETE:**
```properties
Statement Type: DELETE
Table Name: temp_records
Update Keys: record_id
```

### Example Flows

**Example 1: JSON to Database**

```
GetFile (JSON data)
  ↓
PutDatabaseRecord
  Record Reader: JsonTreeReader
  Statement Type: INSERT
  Table Name: customers
```

**Input JSON:**
```json
[
  {
    "customer_id": 1001,
    "name": "John Doe",
    "email": "john@example.com",
    "status": "active",
    "created_date": "2024-01-15"
  },
  {
    "customer_id": 1002,
    "name": "Jane Smith",
    "email": "jane@example.com",
    "status": "active",
    "created_date": "2024-01-15"
  }
]
```

**Example 2: CSV to Database**

```
GetFile (CSV file)
  ↓
UpdateAttribute
  schema.name: customer_schema
  ↓
PutDatabaseRecord
  Record Reader: CSVReader
  Statement Type: INSERT
  Table Name: customers
```

**Input CSV:**
```csv
customer_id,name,email,status,created_date
1001,John Doe,john@example.com,active,2024-01-15
1002,Jane Smith,jane@example.com,active,2024-01-15
```

**Example 3: UPSERT Pattern**

```
ConsumeKafka (real-time updates)
  ↓
ConvertRecord (Avro → JSON)
  ↓
PutDatabaseRecord
  Statement Type: UPSERT
  Table Name: product_inventory
  Update Keys: product_id
```

## QueryDatabaseTable Processor

### Overview

Incrementally fetches data from database tables using maximum value tracking.

### Configuration

```properties
Database Connection Pooling Service: DBCPConnectionPool
Database Type: MySQL
Table Name: orders
Columns to Return: (leave empty for all)
Maximum-value Columns: order_id
Where Clause: status = 'pending'
```

### How It Works

**First Run:**
```sql
SELECT * FROM orders 
WHERE status = 'pending'
ORDER BY order_id
```

Tracks `max(order_id) = 1000`

**Second Run:**
```sql
SELECT * FROM orders 
WHERE status = 'pending' 
  AND order_id > 1000
ORDER BY order_id
```

**State Management:**
- Stores maximum value in NiFi state
- Only fetches new/updated records
- Prevents duplicate processing

### Multiple Maximum-Value Columns

```properties
Maximum-value Columns: updated_at, order_id
```

Generates:
```sql
SELECT * FROM orders
WHERE (updated_at > '2024-01-15 10:00:00')
   OR (updated_at = '2024-01-15 10:00:00' AND order_id > 1000)
ORDER BY updated_at, order_id
```

### Complete Incremental Sync Flow

```
QueryDatabaseTable
  Table Name: orders
  Maximum-value Columns: updated_at
  Run Schedule: 0 */5 * * * ? (every 5 minutes)
  ↓
ConvertAvroToJSON
  ↓
QueryRecord (filter new orders)
  ↓
PutElasticsearch
  ↓
UpdateAttribute
  sync.timestamp: ${now()}
  records.processed: ${fragment.count}
```

## QueryDatabaseTableRecord Processor

### Overview

Record-based version of QueryDatabaseTable with configurable output format.

### Configuration

```properties
Database Connection Pooling Service: DBCPConnectionPool
Table Name: customers
Maximum-value Columns: customer_id
Record Writer: JsonRecordSetWriter
```

**Advantages over QueryDatabaseTable:**
- Choose output format (JSON, CSV, Avro)
- Better schema handling
- More efficient for large result sets

## GenerateTableFetch Processor

### Overview

Generates SQL queries for partitioned/parallel table fetching.

### Configuration

```properties
Database Connection Pooling Service: DBCPConnectionPool
Table Name: large_table
Partition Size: 10000
Column Names: (leave empty for all)
Maximum-value Columns: id
```

### How It Works

**For table with 50,000 rows:**

Generates 5 FlowFiles with queries:

FlowFile 1:
```sql
SELECT * FROM large_table WHERE id >= 1 AND id < 10001 ORDER BY id
```

FlowFile 2:
```sql
SELECT * FROM large_table WHERE id >= 10001 AND id < 20001 ORDER BY id
```

... and so on.

### Parallel Processing Flow

```
GenerateTableFetch
  Partition Size: 10000
  ↓
ExecuteSQLRecord (Concurrent Tasks: 5)
  ↓
MergeContent (optional - combine results)
  ↓
PutDatabaseRecord (load to warehouse)
```

## Common Database Patterns

### Pattern 1: Database to Elasticsearch

**Real-time sync of database changes**

```
QueryDatabaseTableRecord
  Table Name: products
  Maximum-value Columns: updated_at
  Run Schedule: 0 * * * * ? (every minute)
  Record Writer: JsonRecordSetWriter
  ↓
UpdateAttribute
  elasticsearch.index: products
  elasticsearch.id: ${product_id}
  ↓
PutElasticsearch
  Index: ${elasticsearch.index}
  Identifier: ${elasticsearch.id}
  Index Operation: Index
```

### Pattern 2: Database Replication

**Replicate table from MySQL to PostgreSQL**

```
QueryDatabaseTable (MySQL)
  Database Connection: MySQL-Pool
  Table Name: orders
  Maximum-value Columns: order_id
  ↓
ConvertAvroToJSON
  ↓
PutDatabaseRecord (PostgreSQL)
  Database Connection: PostgreSQL-Pool
  Statement Type: UPSERT
  Table Name: orders
  Update Keys: order_id
```

### Pattern 3: Data Warehouse ETL

**Extract, transform, load to data warehouse**

```
ExecuteSQLRecord (Production DB)
  SQL: SELECT 
         o.order_id,
         o.order_date,
         c.customer_name,
         SUM(oi.quantity * oi.price) AS total
       FROM orders o
       JOIN customers c ON o.customer_id = c.customer_id
       JOIN order_items oi ON o.order_id = oi.order_id
       WHERE o.order_date >= CURRENT_DATE - 1
       GROUP BY o.order_id, o.order_date, c.customer_name
  ↓
QueryRecord (add dimensions)
  SQL: SELECT 
         *,
         EXTRACT(YEAR FROM order_date) AS year,
         EXTRACT(MONTH FROM order_date) AS month,
         EXTRACT(DAY FROM order_date) AS day
       FROM FLOWFILE
  ↓
PutDatabaseRecord (Data Warehouse)
  Table Name: fact_orders
  Statement Type: INSERT
```

### Pattern 4: Database Archival

**Archive old records to cheaper storage**

```
ExecuteSQLRecord
  SQL: SELECT * FROM transactions 
       WHERE created_date < CURRENT_DATE - INTERVAL '365' DAY
  Record Writer: ParquetRecordSetWriter
  ↓
PutHDFS (or PutS3Object)
  Directory: /archive/${now():format('yyyy')}/${now():format('MM')}
  ↓ success
PutSQL (delete archived records)
  SQL: DELETE FROM transactions 
       WHERE created_date < CURRENT_DATE - INTERVAL '365' DAY
       AND transaction_id IN (${archived.ids})
```

### Pattern 5: Database Validation

**Compare data between source and target databases**

```
Flow 1 (Source):
ExecuteSQLRecord
  SQL: SELECT * FROM customers
  ↓
UpdateAttribute
  source: production
  ↓
[Comparison Point]

Flow 2 (Target):
ExecuteSQLRecord
  SQL: SELECT * FROM customers
  ↓
UpdateAttribute
  source: replica
  ↓
[Comparison Point]

[Comparison Point]:
ExecuteScript (Python - compare records)
  ↓ differences
LogAttribute
  ↓
PutFile (discrepancy report)
```

### Pattern 6: Change Data Capture (CDC)

**Track changes using audit columns**

```
QueryDatabaseTableRecord
  Table Name: customer_updates
  Maximum-value Columns: updated_at, customer_id
  Run Schedule: 0 */5 * * * ? (every 5 min)
  ↓
RouteOnAttribute
  ${operation:equals('INSERT')}
  ${operation:equals('UPDATE')}
  ${operation:equals('DELETE')}
  ↓ INSERT
PutKafka (topic: customer.insert)
  ↓ UPDATE  
PutKafka (topic: customer.update)
  ↓ DELETE
PutKafka (topic: customer.delete)
```

## Transaction Management

### Enabling Transactions

```properties
PutDatabaseRecord:
  Support Fragmented Transactions: true
  Transaction Timeout: 0 sec (no timeout)
  Rollback On Failure: true
```

### Transaction Flow Example

```
GetFile (batch of records)
  ↓
SplitRecord (split into chunks)
  ↓
PutDatabaseRecord
  Support Fragmented Transactions: true
  Rollback On Failure: true
  ↓ success (all chunks succeed)
DeleteFile (source file)
  
  ↓ failure (any chunk fails)
PutFile (error directory)
  # Entire transaction rolled back
```

### Manual Transaction Control

```
ExecuteSQL
  SQL: BEGIN TRANSACTION;
  ↓
PutSQL (operation 1)
  ↓ success
PutSQL (operation 2)
  ↓ success
ExecuteSQL
  SQL: COMMIT;
  
  ↓ failure (any step)
ExecuteSQL
  SQL: ROLLBACK;
  ↓
LogAttribute
  ↓
[Error handling]
```

## Performance Optimization

### 1. Connection Pooling

```properties
DBCPConnectionPool:
  Max Total Connections: 20
  Max Wait Time: 5000 millis
  Min Idle: 5
  Max Idle: 10
  
  # Enable prepared statement caching
  Max Open Prepared Statements: 50
```

### 2. Batch Processing

```properties
PutDatabaseRecord:
  Batch Size: 1000  # Commit every 1000 records
  
PutSQL:
  Batch Size: 100   # Execute 100 statements per batch
```

### 3. Partitioned Fetching

For large tables:

```
GenerateTableFetch
  Partition Size: 50000
  ↓
ExecuteSQLRecord
  Concurrent Tasks: 10  # Process 10 partitions in parallel
```

### 4. Index Usage

Ensure Maximum-value Columns are indexed:

```sql
-- Create index for incremental queries
CREATE INDEX idx_orders_updated 
ON orders(updated_at, order_id);

-- Verify index usage
EXPLAIN SELECT * FROM orders 
WHERE updated_at > '2024-01-01' 
ORDER BY updated_at;
```

### 5. Query Optimization

**Bad Query (Full table scan):**
```sql
SELECT * FROM orders 
WHERE YEAR(order_date) = 2024
```

**Good Query (Uses index):**
```sql
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01'
```

### 6. Limit Result Sets

```properties
ExecuteSQLRecord:
  Max Rows Per Flow File: 10000
  
  SQL: SELECT * FROM large_table 
       LIMIT 10000 OFFSET ${offset}
```

## Security Best Practices

### 1. Encrypted Credentials

Use NiFi's sensitive property encryption:

```properties
DBCPConnectionPool:
  Password: [encrypted value]
  # Automatically encrypted by NiFi
```

### 2. Least Privilege

Create dedicated database users:

```sql
-- Read-only user for queries
CREATE USER 'nifi_read'@'%' IDENTIFIED BY 'secure_password';
GRANT SELECT ON mydb.* TO 'nifi_read'@'%';

-- Write user with limited permissions
CREATE USER 'nifi_write'@'%' IDENTIFIED BY 'secure_password';
GRANT SELECT, INSERT, UPDATE ON mydb.orders TO 'nifi_write'@'%';
```

### 3. SSL/TLS Connections

**MySQL:**
```properties
URL: jdbc:mysql://localhost:3306/mydb?useSSL=true&requireSSL=true&verifyServerCertificate=true
```

**PostgreSQL:**
```properties
URL: jdbc:postgresql://localhost:5432/mydb?ssl=true&sslmode=require
```

**SQL Server:**
```properties
URL: jdbc:sqlserver://localhost:1433;databaseName=mydb;encrypt=true;trustServerCertificate=false;
```

### 4. Connection Validation

```properties
DBCPConnectionPool:
  Validation Query: SELECT 1
  Validation Timeout: 5 sec
  Test On Borrow: true
  Test While Idle: true
```

### 5. SQL Injection Prevention

**Always use parameterized queries:**

✅ **Good:**
```
ExecuteSQL:
  SQL: SELECT * FROM users WHERE username = ?
  SQL Arguments: ${username}
```

❌ **Bad (SQL Injection vulnerable):**
```
ExecuteSQL:
  SQL: SELECT * FROM users WHERE username = '${username}'
```

## Error Handling

### Retry Logic

```
QueryDatabaseTable
  ↓ failure
UpdateAttribute
  retry.count: ${retry.count:plus(1)}
  ↓
RouteOnAttribute
  ${retry.count:toNumber():lt(3)}
  ↓ true (retry)
Wait (5 sec)
  ↓
[Loop back to QueryDatabaseTable]
  
  ↓ false (max retries)
LogAttribute
  ↓
PutFile (dead letter queue)
  ↓
InvokeHTTP (alert)
```

### Connection Failure Handling

```
ExecuteSQLRecord
  ↓ failure
LogAttribute (log connection error)
  ↓
RouteOnAttribute
  error.type: ${exception:contains('ConnectionException')}
  ↓ true
UpdateAttribute
  alert.severity: HIGH
  alert.message: Database connection failed
  ↓
InvokeHTTP (ops webhook)
  ↓
PutFile (queue for retry)
```

### Data Quality Checks

```
ExecuteSQLRecord
  ↓
ValidateRecord (schema validation)
  ↓ valid
QueryRecord (business rules)
  SQL: SELECT * FROM FLOWFILE 
       WHERE email LIKE '%@%.%'
         AND age > 0
  ↓ matched
PutDatabaseRecord
  
  ↓ unmatched (from QueryRecord)
LogAttribute
  ↓
PutFile (invalid records)
```

## Monitoring and Logging

### Query Performance Monitoring

```
ExecuteSQLRecord
  ↓
UpdateAttribute (before query)
  query.start: ${now()}
  ↓
[Execute query]
  ↓
UpdateAttribute (after query)
  query.end: ${now()}
  query.duration: ${query.end:minus(${query.start})}
  query.record.count: ${fragment.count}
  ↓
RouteOnAttribute
  ${query.duration:toNumber():gt(30000)}  # > 30 seconds
  ↓ true (slow query)
LogAttribute (alert slow query)
```

### Connection Pool Monitoring

```
ExecuteScript (scheduled - every 5 min)
  Language: Groovy
  Script:
```

```groovy
import org.apache.commons.dbcp2.BasicDataSource

def dbcpService = context.controllerServiceLookup
    .getControllerService("DBCPConnectionPool")

def dataSource = dbcpService.getDataSource()

def metrics = [
    active: dataSource.getNumActive(),
    idle: dataSource.getNumIdle(),
    max: dataSource.getMaxTotal(),
    wait_count: dataSource.getMaxWaitMillis()
]

def flowFile = session.create()
flowFile = session.write(flowFile) { outputStream ->
    outputStream.write(groovy.json.JsonOutput.toJson(metrics).bytes)
}

session.transfer(flowFile, REL_SUCCESS)
```

### Audit Logging

```
PutDatabaseRecord
  ↓ success
UpdateAttribute
  audit.operation: INSERT
  audit.table: ${table.name}
  audit.record.count: ${record.count}
  audit.timestamp: ${now()}
  audit.user: ${username}
  ↓
ExecuteSQL (log to audit table)
  SQL: INSERT INTO audit_log 
       (operation, table_name, record_count, timestamp, username)
       VALUES (?, ?, ?, ?, ?)
  SQL Arguments: ${audit.operation}, ${audit.table}, 
                 ${audit.record.count}, ${audit.timestamp}, ${audit.user}
```

## Testing Database Flows

### 1. Use Test Database

```properties
Development DBCPConnectionPool:
  URL: jdbc:mysql://localhost:3306/test_db
  User: test_user
  
Production DBCPConnectionPool:
  URL: jdbc:mysql://prod-db:3306/production_db
  User: prod_user
```

### 2. Sample Data Generation

```
GenerateFlowFile
  Custom Text:
```

```json
[
  {"id": 1, "name": "Test User 1", "email": "test1@example.com"},
  {"id": 2, "name": "Test User 2", "email": "test2@example.com"}
]
```

```
  ↓
PutDatabaseRecord (test database)
  ↓
ExecuteSQLRecord (verify insert)
  SQL: SELECT * FROM test_table WHERE id IN (1, 2)
  ↓
LogAttribute (verify results)
```

### 3. Dry Run Mode

```
ExecuteSQLRecord
  ↓
PutFile (preview SQL statements)
  # Don't execute, just save generated SQL
```

## Advanced Patterns

### Pattern 1: Multi-Database Join

**Join data from MySQL and PostgreSQL:**

```
Flow 1 (MySQL):
ExecuteSQLRecord (MySQL)
  SQL: SELECT order_id, customer_id, total 
       FROM orders 
       WHERE order_date = CURRENT_DATE
  ↓
UpdateAttribute
  source: mysql
  join.key: ${customer_id}
  ↓
[Join Point]

Flow 2 (PostgreSQL):
ExecuteSQLRecord (PostgreSQL)
  SQL: SELECT customer_id, name, tier 
       FROM customers
  ↓
UpdateAttribute
  source: postgres
  join.key: ${customer_id}
  ↓
[Join Point]

[Join Point]:
ExecuteScript (Python - join on customer_id)
  ↓
PutElasticsearch (combined data)
```

### Pattern 2: Database Sharding

**Read from multiple sharded databases:**

```
GenerateFlowFile (shard list)
  Custom Text: shard1,shard2,shard3
  ↓
SplitText (one FlowFile per shard)
  ↓
UpdateAttribute
  db.shard: ${line}
  db.url: jdbc:mysql://db-${db.shard}:3306/mydb
  ↓
ExecuteSQLRecord
  SQL: SELECT * FROM users 
       WHERE user_id % 3 = ${db.shard}
  ↓
MergeContent (combine all shards)
  ↓
PutDatabaseRecord (central warehouse)
```

### Pattern 3: Slowly Changing Dimensions (SCD Type 2)

**Track historical changes:**

```
QueryDatabaseTableRecord (source)
  Table Name: customers
  Maximum-value Columns: updated_at
  ↓
ExecuteSQLRecord (check if exists in warehouse)
  SQL: SELECT * FROM dim_customer 
       WHERE customer_id = ${customer_id} 
         AND is_current = 1
  ↓
RouteOnAttribute
  ${record.count:gt(0)}
  ↓ true (exists - update)
ExecuteSQL (close old record)
  SQL: UPDATE dim_customer 
       SET is_current = 0, 
           end_date = CURRENT_DATE 
       WHERE customer_id = ? 
         AND is_current = 1
  SQL Arguments: ${customer_id}
  ↓
PutDatabaseRecord (insert new version)
  Table Name: dim_customer
  # Adds: is_current=1, start_date=CURRENT_DATE, end_date=NULL
  
  ↓ false (from RouteOnAttribute - doesn't exist)
PutDatabaseRecord (insert new)
  Table Name: dim_customer
```

### Pattern 4: Database Health Check

```
GenerateFlowFile (scheduled every 1 min)
  ↓
ExecuteSQL (connectivity test)
  SQL: SELECT 1 AS health_check
  ↓ success
UpdateAttribute
  db.status: UP
  db.check.timestamp: ${now()}
  ↓
[Continue monitoring]
  
  ↓ failure
UpdateAttribute
  db.status: DOWN
  db.check.timestamp: ${now()}
  alert.severity: CRITICAL
  ↓
InvokeHTTP (alert PagerDuty)
  ↓
PutFile (incident log)
```

## Common Issues and Solutions

### Issue 1: Driver Not Found

**Error:** `java.lang.ClassNotFoundException: com.mysql.jdbc.Driver`

**Solution:**
1. Verify JAR file exists in specified location
2. Restart NiFi after adding new drivers
3. Check driver class name spelling
4. Ensure JAR is not corrupted

```bash
# Verify JAR
jar -tf mysql-connector-java-8.0.33.jar | grep Driver

# Should show:
# com/mysql/cj/jdbc/Driver.class
```

### Issue 2: Connection Timeout

**Error:** `Connection timeout after 30000 ms`

**Solution:**
```properties
DBCPConnectionPool:
  Max Wait Time: 60000 millis
  Validation Timeout: 10 sec
  
  # Check firewall/network
  # Verify database is running
  # Confirm connection URL is correct
```

### Issue 3: Too Many Connections

**Error:** `Too many connections`

**Solution:**
```properties
DBCPConnectionPool:
  Max Total Connections: 10  # Reduce from 20
  
# Or increase database max_connections
# MySQL:
SET GLOBAL max_connections = 200;
```

### Issue 4: Deadlocks

**Error:** `Deadlock found when trying to get lock`

**Solution:**
```
PutDatabaseRecord:
  Batch Size: 100  # Smaller batches
  Rollback On Failure: true
  
  # Implement retry logic
  ↓ retry
Wait (Random 1-5 sec)
  ↓
[Retry operation]
```

### Issue 5: Slow Queries

**Solution:**
```sql
-- Add indexes
CREATE INDEX idx_orders_date ON orders(order_date);

-- Analyze query plan
EXPLAIN SELECT * FROM orders WHERE order_date > '2024-01-01';

-- Partition large tables
ALTER TABLE orders 
PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);
```

## Summary

Database connectivity in NiFi enables:
- ✅ Execute queries with ExecuteSQL/ExecuteSQLRecord
- ✅ Insert/update data with PutSQL/PutDatabaseRecord
- ✅ Incremental sync with QueryDatabaseTable
- ✅ Parallel fetching with GenerateTableFetch
- ✅ Multiple database support via JDBC
- ✅ Transaction management

Key Takeaways:
1. Use connection pooling for efficiency
2. Implement incremental syncing for large tables
3. Use parameterized queries to prevent SQL injection
4. Monitor connection pool and query performance
5. Handle errors gracefully with retry logic
6. Test with sample data before production

```
## 10_wait_notify.md

```markdown
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
## 11_kafka_integration.md

```markdown
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
## 12_tailfile_logs.md

```markdown
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
## 14_rest_api.md

```markdown
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
