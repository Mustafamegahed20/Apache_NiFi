## 09_database_connections.md

```markdown
```
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
