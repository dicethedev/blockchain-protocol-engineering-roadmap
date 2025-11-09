# RPC Systems - Remote Procedure Call Protocols

## Overview
Master RPC (Remote Procedure Call) systems - a fundamental building block for distributed systems and blockchain protocols. Document everything you learn about RPC design, implementation, and optimization.

**Your Experience:** Built DiceRPC - a complete JSON-RPC 2.0 framework in Rust

---

## Learning Objectives
- [x] Understand RPC fundamentals and design patterns
- [x] Implement JSON-RPC 2.0 specification (DiceRPC)
- [x] Master transport layer abstractions
- [ ] Learn gRPC and Protocol Buffers
- [x] Handle concurrent requests safely
- [x] Design clean RPC APIs

---

## What is RPC?

### Core Concept
```
Remote Procedure Call allows a program to execute code on another computer
as if it were a local function call.

Local function call:
    result = add(5, 3)  // Executes locally

RPC call:
    result = remote.add(5, 3)  // Executes on remote server

The complexity of network communication is hidden from the developer.
```

**Key Insight from DiceRPC:**
```
While RPC looks simple from the outside (just a function call),
there's significant complexity under the hood:
- Serialization (Rust structs → JSON)
- Network transport (TCP/HTTP)
- Framing (message boundaries)
- Error handling (network failures, timeouts)
- Concurrency (multiple clients)
- Authentication (who can call what)

All this complexity must be transparent to the user!
```

### RPC vs REST vs GraphQL

**Learned from building DiceRPC:**

| Aspect | RPC | REST | GraphQL |
|--------|-----|------|---------|
| **Abstraction** | Function calls | Resources | Queries |
| **Protocol** | JSON-RPC, gRPC | HTTP verbs | HTTP POST |
| **Data Format** | JSON, Protobuf | JSON, XML | JSON |
| **Type Safety** | Depends | No | Yes (schema) |
| **Overhead** | Low | Medium | Medium |
| **Use Case** | Internal services | Public APIs | Data aggregation |

**When to use RPC (DiceRPC taught me):**
- Internal microservices communication
- High-performance requirements
- Type-safe communication desired
- Action-oriented APIs (not resource-oriented)
- Blockchain node communication

**Trade-offs:**
```
Pros:
+ Simple mental model (just call functions)
+ Efficient (minimal overhead)
+ Type-safe (with right implementation)
+ Fast (binary protocols like gRPC)

Cons:
- Tighter coupling than REST
- Harder to debug (binary formats)
- Less tooling than REST
- Requires client library
```

---

## JSON-RPC 2.0 Specification

### The Protocol (What I Implemented in DiceRPC)

**Request Format:**
```json
{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": [42, 23],
  "id": 1
}
```

**Response Format:**
```json
{
  "jsonrpc": "2.0",
  "result": 19,
  "id": 1
}
```

**Error Format:**
```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,
    "message": "Method not found",
    "data": "Method 'substract' does not exist"
  },
  "id": 1
}
```

### Critical Implementation Details

#### 1. Version Field is Required
```rust
// DiceRPC implementation
#[derive(Serialize, Deserialize)]
struct Request {
    jsonrpc: String,  // MUST be "2.0"
    method: String,
    params: Option<Value>,
    id: Option<Value>,
}

// Validation
fn validate_request(req: &Request) -> Result<()> {
    if req.jsonrpc != "2.0" {
        return Err(RpcError::InvalidRequest);
    }
    Ok(())
}
```

**Why it matters:**
- Allows protocol evolution
- Clients can support multiple versions
- Clear version negotiation

#### 2. ID Must Be Echoed Back
```rust
// From DiceRPC
async fn handle_request(request: Request) -> Response {
    // Process the request...
    
    Response {
        jsonrpc: "2.0".to_string(),
        result: Some(result),
        error: None,
        id: request.id,  // ⚠️ MUST echo back exactly!
    }
}
```

**Critical lesson:**
- ID can be number, string, or null
- Must return EXACT same ID (even if it's weird)
- Allows client to match responses to requests
- Essential for async/concurrent requests

#### 3. Notifications Have No ID
```json
// Notification - no response expected
{
  "jsonrpc": "2.0",
  "method": "notify",
  "params": ["hello"]
  // No "id" field!
}
```

**Implementation in DiceRPC:**
```rust
if request.id.is_none() {
    // This is a notification
    // Execute but don't send response
    handle_method(&request.method, request.params).await?;
    return Ok(None);  // No response!
}
```

#### 4. Params Can Be Array or Object
```json
// Array params (positional)
{
  "method": "subtract",
  "params": [42, 23]
}

// Object params (named)
{
  "method": "subtract", 
  "params": {"minuend": 42, "subtrahend": 23}
}

// No params
{
  "method": "get_version"
  // params is omitted entirely
}
```

**DiceRPC pattern:**
```rust
use serde_json::Value;

fn handle_method(method: &str, params: Option<Value>) -> Result<Value> {
    match method {
        "add" => {
            // Extract params flexibly
            let params = params.ok_or(RpcError::InvalidParams)?;
            
            // Handle both array and object
            let (a, b) = match params {
                Value::Array(arr) => (
                    arr[0].as_i64().ok_or(RpcError::InvalidParams)?,
                    arr[1].as_i64().ok_or(RpcError::InvalidParams)?,
                ),
                Value::Object(obj) => (
                    obj["a"].as_i64().ok_or(RpcError::InvalidParams)?,
                    obj["b"].as_i64().ok_or(RpcError::InvalidParams)?,
                ),
                _ => return Err(RpcError::InvalidParams),
            };
            
            Ok(json!(a + b))
        }
        _ => Err(RpcError::MethodNotFound)
    }
}
```

#### 5. Batch Requests
```json
// Send multiple requests at once
[
  {"jsonrpc": "2.0", "method": "add", "params": [1,2], "id": 1},
  {"jsonrpc": "2.0", "method": "subtract", "params": [5,3], "id": 2}
]

// Response is also array
[
  {"jsonrpc": "2.0", "result": 3, "id": 1},
  {"jsonrpc": "2.0", "result": 2, "id": 2}
]
```

**DiceRPC implementation:**
```rust
async fn handle_batch(requests: Vec<Request>) -> Vec<Response> {
    // Process concurrently
    let handles: Vec<_> = requests
        .into_iter()
        .enumerate()
        .map(|(idx, req)| {
            tokio::spawn(async move {
                let response = handle_single(req).await;
                (idx, response)
            })
        })
        .collect();
    
    // Collect and maintain order
    let mut results: Vec<_> = futures::future::join_all(handles)
        .await
        .into_iter()
        .map(|r| r.unwrap())
        .collect();
    
    results.sort_by_key(|(idx, _)| *idx);
    results.into_iter().map(|(_, resp)| resp).collect()
}
```

**Key insight:**
- Batch requests can be processed concurrently
- But responses MUST maintain order
- Great for performance with multiple calls

#### 6. Standard Error Codes

**From the Spec (implemented in DiceRPC):**
```rust
// Standard error codes
const PARSE_ERROR: i32 = -32700;      // Invalid JSON
const INVALID_REQUEST: i32 = -32600;  // Not valid Request object  
const METHOD_NOT_FOUND: i32 = -32601; // Method doesn't exist
const INVALID_PARAMS: i32 = -32602;   // Invalid params
const INTERNAL_ERROR: i32 = -32603;   // Internal JSON-RPC error

// Server error range: -32000 to -32099 (custom)
const SERVER_ERROR: i32 = -32000;

#[derive(Error, Debug)]
pub enum RpcError {
    #[error("Parse error")]
    ParseError,  // Code: -32700
    
    #[error("Invalid request")]
    InvalidRequest,  // Code: -32600
    
    #[error("Method not found: {0}")]
    MethodNotFound(String),  // Code: -32601
    
    #[error("Invalid params")]
    InvalidParams,  // Code: -32602
    
    #[error("Internal error: {0}")]
    InternalError(String),  // Code: -32603
}

impl RpcError {
    fn error_code(&self) -> i32 {
        match self {
            RpcError::ParseError => PARSE_ERROR,
            RpcError::InvalidRequest => INVALID_REQUEST,
            RpcError::MethodNotFound(_) => METHOD_NOT_FOUND,
            RpcError::InvalidParams => INVALID_PARAMS,
            RpcError::InternalError(_) => INTERNAL_ERROR,
        }
    }
}
```

---

## Transport Layer

### TCP vs HTTP (Both Implemented in DiceRPC)

**TCP Transport:**
```rust
// From DiceRPC - TCP with length-prefixed framing

async fn handle_tcp_connection(mut stream: TcpStream) -> Result<()> {
    loop {
        // Read 4-byte length prefix
        let len = stream.read_u32().await? as usize;
        
        // Read exact message
        let mut buffer = vec![0u8; len];
        stream.read_exact(&mut buffer).await?;
        
        // Parse JSON-RPC request
        let request: Request = serde_json::from_slice(&buffer)?;
        
        // Handle request
        let response = handle_request(request).await?;
        
        // Serialize response
        let response_bytes = serde_json::to_vec(&response)?;
        
        // Write length + response
        stream.write_u32(response_bytes.len() as u32).await?;
        stream.write_all(&response_bytes).await?;
    }
}
```

**HTTP Transport:**
```rust
// From DiceRPC - HTTP is simpler (built-in framing)

async fn handle_http_request(req: HttpRequest) -> HttpResponse {
    // Extract body
    let body = req.body();
    
    // Parse JSON-RPC
    let request: Request = serde_json::from_slice(&body)?;
    
    // Handle
    let response = handle_request(request).await?;
    
    // Return HTTP response
    HttpResponse::Ok()
        .content_type("application/json")
        .body(serde_json::to_string(&response)?)
}
```

### Why Length-Prefixed Framing? (Critical Learning)

**The Problem:**
```
TCP is a STREAM, not a MESSAGE protocol.

Data sent: "Message1Message2Message3"
Could arrive as:
- Read 1: "Message1Mes"
- Read 2: "sage2Message3"

How do you know where messages start and end?
```

**Solutions Compared:**

| Method | Pros | Cons | Verdict |
|--------|------|------|---------|
| **Newline delimiter** | Simple | Breaks with formatted JSON | ❌ Don't use |
| **Fixed size** | Very simple | Inflexible, wastes space | ❌ Don't use |
| **Special delimiter** | Simple | Escaping needed | ⚠️ Complex |
| **Length prefix** | Reliable, efficient | Slightly more complex | ✅ Best choice |

**Length-Prefix Implementation (DiceRPC):**
```
Message Format:
┌──────────────┬────────────────────────────┐
│ Length (4B)  │  JSON Payload (Length B)  │
│  Big-endian  │                            │
└──────────────┴────────────────────────────┘

Example:
┌──────────────┬────────────────────────────┐
│  0x0000001A  │  {"jsonrpc":"2.0",...}    │
│   (26 bytes) │     (26 bytes)             │
└──────────────┴────────────────────────────┘
```

**Code Pattern:**
```rust
// Write with framing
async fn write_framed(stream: &mut TcpStream, data: &[u8]) -> Result<()> {
    // Length prefix (u32, big-endian)
    stream.write_u32(data.len() as u32).await?;
    // Payload
    stream.write_all(data).await?;
    stream.flush().await?;
    Ok(())
}

// Read with framing
async fn read_framed(stream: &mut TcpStream) -> Result<Vec<u8>> {
    // Read length
    let len = stream.read_u32().await? as usize;
    
    // Validate length (prevent DoS)
    if len > MAX_MESSAGE_SIZE {
        return Err(Error::MessageTooLarge);
    }
    
    // Read exact payload
    let mut buffer = vec![0u8; len];
    stream.read_exact(&mut buffer).await?;
    
    Ok(buffer)
}
```

**Why Big-Endian (Network Byte Order):**
```
Network protocols use big-endian by convention.
This ensures compatibility across different architectures.

Little-endian: 0x12345678 → [78 56 34 12]
Big-endian:   0x12345678 → [12 34 56 78] ✅
```

---

## ⚡ Concurrency & Performance

### Task-Per-Connection Pattern (DiceRPC)

**Architecture:**
```rust
// Main server loop
async fn run_server(addr: &str) -> Result<()> {
    let listener = TcpListener::bind(addr).await?;
    let handler = Arc::new(MyHandler::new());
    
    println!("Listening on {}", addr);
    
    loop {
        let (stream, addr) = listener.accept().await?;
        let handler = handler.clone();  // Clone Arc (cheap!)
        
        // Spawn independent task for this connection
        tokio::spawn(async move {
            println!("New connection from {}", addr);
            
            if let Err(e) = handle_connection(stream, handler).await {
                eprintln!("Connection error: {}", e);
            }
            
            println!("Connection closed: {}", addr);
        });
    }
}
```

**Key Insights:**

1. **Arc for Shared State:**
```rust
// ❌ Wrong - This won't compile
let handler = MyHandler::new();
tokio::spawn(async move {
    handler.process();  // Error: handler moved into first task
});

// ✅ Correct - Arc allows sharing
let handler = Arc::new(MyHandler::new());
let handler_clone = handler.clone();  // Increment ref count
tokio::spawn(async move {
    handler_clone.process();  // Works!
});
```

2. **Why tokio::spawn:**
```
- Each connection runs independently
- Blocking one connection doesn't block others
- Automatic work-stealing between CPU cores
- Graceful handling of connection failures
```

3. **Performance Impact:**
```
Sequential (no spawning):
  Connection 1: |====== 100ms ======|
  Connection 2:                      |====== 100ms ======|
  Total: 200ms for 2 connections

Concurrent (with tokio::spawn):
  Connection 1: |====== 100ms ======|
  Connection 2: |====== 100ms ======|
  Total: 100ms for 2 connections
  
DiceRPC measured: ~50x improvement with concurrent connections
```

### Shared Mutable State

**The Challenge:**
```
Multiple connections need to share state.
But Rust won't let you mutate shared data!

// This doesn't work
let counter = 0;
tokio::spawn(async move {
    counter += 1;  // Error: cannot mutate
});
```

**DiceRPC Solution:**
```rust
use std::sync::Arc;
use tokio::sync::Mutex;

#[derive(Clone)]
struct SharedState {
    counter: Arc<Mutex<u64>>,
}

impl SharedState {
    fn new() -> Self {
        Self {
            counter: Arc::new(Mutex::new(0)),
        }
    }
    
    async fn increment(&self) {
        let mut counter = self.counter.lock().await;
        *counter += 1;
        // Lock automatically released when `counter` goes out of scope
    }
    
    async fn get(&self) -> u64 {
        let counter = self.counter.lock().await;
        *counter
    }
}

// Usage
let state = SharedState::new();
tokio::spawn({
    let state = state.clone();
    async move {
        state.increment().await;
    }
});
```

**Pattern Breakdown:**
```
Arc<Mutex<T>>

Arc = Atomic Reference Counting
  - Allows sharing across threads/tasks
  - Clone increases ref count
  - Drop decreases ref count
  - Data freed when count reaches 0

Mutex = Mutual Exclusion
  - Ensures only one task accesses data at a time
  - .lock().await acquires lock
  - Lock released when guard drops
  
Together: Safe shared mutable state!
```

**Performance Considerations:**
```rust
// ❌ Bad - Lock held too long
async fn bad_pattern(state: &SharedState) {
    let mut data = state.data.lock().await;
    // ... lots of work ...
    expensive_computation().await;  // Lock still held!
    *data += 1;
}

// ✅ Good - Minimize lock duration
async fn good_pattern(state: &SharedState) {
    let result = expensive_computation().await;
    
    // Lock only when necessary
    {
        let mut data = state.data.lock().await;
        *data += result;
    }  // Lock released here
}
```

---

## DiceRPC Performance Results

### Benchmarks
```
Baseline (sequential):
  Throughput: ~100 req/sec
  Latency p50: 10ms
  Latency p99: 15ms

With concurrency (tokio::spawn):
  Throughput: ~5,000 req/sec  (50x improvement!)
  Latency p50: 0.5ms
  Latency p99: 5ms

With optimizations:
  Throughput: ~8,000 req/sec
  Latency p50: 0.3ms
  Latency p99: 3ms
```

---

## Key Takeaways from Building DiceRPC

### What I Mastered

**1. RPC Fundamentals:**
- How RPC abstracts network communication
- Request/response patterns
- Error handling across network
- Specification compliance

**2. Transport Layer:**
- Length-prefixed framing for TCP
- Why framing is necessary
- HTTP vs TCP trade-offs
- Transport abstraction patterns

**3. Concurrency:**
- Task-per-connection pattern
- Arc for shared ownership
- Mutex for safe mutation
- Lock granularity matters

**4. Protocol Design:**
- Spec compliance is critical
- Versioning enables evolution
- Error codes must be standardized
- Batch operations improve performance

**5. Production Concerns:**
- Authentication (API keys, JWT)
- Rate limiting
- Timeout handling
- Graceful shutdown
- Observability (logging, metrics)

### Biggest Challenges Solved

1. **TCP Framing** - Learned that streams need explicit boundaries
2. **Concurrent State** - Mastered Arc<Mutex<T>> pattern
3. **Async Error Handling** - Type-safe errors with context
4. **Batch Processing** - Concurrent execution with ordered responses
5. **Graceful Shutdown** - Clean resource cleanup

### Skills Gained

**Before DiceRPC:** ⭐⭐ (Basic RPC understanding)  
**After DiceRPC:** ⭐⭐⭐⭐ (Can build production RPC systems)

**Transferable to:**
- Blockchain protocols (Ethereum RPC, Bitcoin RPC)
- P2P communication
- Microservices architecture
- Distributed systems

---

## 🔗 Related Topics

**In This Roadmap:**
- [Networking Fundamentals](../01-foundations/networking-fundamentals.md)
- [P2P Protocols](../01-foundations/p2p-protocols.md)
- [API Design](./api-design.md)
- [Protocol Buffers](./protocol-buffers.md)

**In Projects:**
- [DiceRPC Project](../../projects/dice-rpc/README.md)
- [DiceRPC Learnings](../../projects/dice-rpc/learnings.md)
- [DiceRPC Challenges](../../projects/dice-rpc/challenges.md)

**In Code Patterns:**
- [Rust Async Patterns](../../code-patterns/rust/README.md)

---

## Practice Exercises

### Exercise 1: Build Simple JSON-RPC Client 
**Goal:** Create client for DiceRPC  
**Status:** Complete (built with DiceRPC)  
**Learnings:**
- Client-side framing
- Request ID management
- Error handling

### Exercise 2: Implement Method Registry
**Goal:** Dynamic method registration  
**Status:** [ ] Planned for DiceRPC  

### Exercise 3: Add Request Timeout
**Goal:** Timeout long-running requests  
**Status:** [ ] Planned for DiceRPC v0.2  

---

## Next Steps

### For DiceRPC
- [ ] Add comprehensive logging (tracing)
- [ ] Implement request timeouts
- [ ] WebSocket transport
- [ ] Prometheus metrics

### For Learning
- [ ] Study gRPC for comparison
- [ ] Learn Protocol Buffers
- [ ] Analyze Ethereum JSON-RPC API
- [ ] Study Bitcoin Core RPC

---

**Related Project:** [DiceRPC](../../projects/dice-rpc/)  
