## 03_jolt_transformation.md

```markdown
```
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
