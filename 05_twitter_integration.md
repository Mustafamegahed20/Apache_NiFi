## 05_twitter_integration.md

```markdown
```
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
