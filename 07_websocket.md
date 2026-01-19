## 07_websocket.md

```markdown
```
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
