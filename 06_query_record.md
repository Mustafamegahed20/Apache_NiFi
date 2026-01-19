## 06_query_record.md

```markdown
```
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
