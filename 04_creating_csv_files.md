## 04_creating_csv_files.md

```markdown
```
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
