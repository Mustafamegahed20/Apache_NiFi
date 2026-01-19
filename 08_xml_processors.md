## 08_xml_processors.md

```markdown
```
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
