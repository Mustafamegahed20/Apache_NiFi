## 02_first_flow.md

```markdown
```
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
