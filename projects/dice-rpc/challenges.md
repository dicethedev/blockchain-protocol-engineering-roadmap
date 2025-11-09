# DiceRPC - Challenges & Solutions

## Overview
Document all the technical challenges encountered while building DiceRPC and how I solved them.

You can read more from article on [hackMD](https://hackmd.io/@dicethedev/Hkk1uV2Aex)

---

## 🚧 Major Challenges

### Challenge 1: Concurrent Request Handling
**Status:** ✅ Solved  
**Difficulty:** ⭐⭐⭐⭐  

**The Problem:**
```
Multiple clients sending requests simultaneously needed to be handled concurrently
without blocking each other. Each request should be processed independently
while sharing the same handler state safely.

Specific issues:
- How to spawn tasks for each request
- Sharing handler state across tasks
- Preventing race conditions
- Managing task lifecycle
```

**Initial Attempts:**
```
1. I tried using Mutex for every handler call
   - Result: Too much contention, poor performance
   
2. I tried cloning handler for each request
   - Result: Memory overhead, state synchronization issues
```

**Final Solution:**
```rust
// Using Arc for shared ownership and appropriate synchronization
use std::sync::Arc;
use tokio::task;

#[derive(Clone)]
struct SharedHandler {
    state: Arc<Mutex<HandlerState>>,
}

async fn handle_connection(stream: TcpStream, handler: SharedHandler) {
    task::spawn(async move {
        // Each connection runs in its own task
        process_requests(stream, handler).await
    });
}
```

**Key Insights:**
- Arc<Mutex<T>> for shared mutable state
- Clone the Arc, not the data
- Spawn tasks for independent request handling
- Keep critical sections small

**What I Learned:**
- Rust's ownership model prevents data races at compile time
- tokio::spawn creates independent async tasks
- Arc reference counting enables safe sharing
- Lock granularity matters for performance

**Performance Impact:**
```
Before: ~100 req/sec (sequential)
After: ~5000 req/sec (concurrent)
```

**Related Code:**
- `src/server.rs:45-78`
- `src/handler.rs:23-56`

---

### Challenge 2: Length-Prefixed Framing for TCP
**Status:** ✅ Solved  
**Difficulty:** ⭐⭐⭐  

**The Problem:**
```
TCP is a stream protocol - how do you know where one JSON-RPC message
ends and another begins? Without proper framing, messages could be:
- Split across multiple reads
- Concatenated together
- Incomplete

Needed a way to delimit messages reliably.
```

**Why This Was Hard:**
```
- JSON doesn't have a built-in framing mechanism
- Newline delimiters break with pretty-printed JSON
- Need to handle partial reads
- Must handle message boundaries correctly
```

**Research Done:**
```
Studied how other protocols handle this:
- HTTP uses Content-Length header
- Many binary protocols use length prefixes
- Some use special delimiters (problematic)

Decided on: 4-byte length prefix (u32 big-endian)
```

**Solution:**
```rust
use tokio::io::{AsyncReadExt, AsyncWriteExt};

// Writing a message
async fn write_message(stream: &mut TcpStream, data: &[u8]) -> Result<()> {
    // Write length prefix (4 bytes, big-endian)
    let len = data.len() as u32;
    stream.write_u32(len).await?;
    
    // Write actual message
    stream.write_all(data).await?;
    
    Ok(())
}

// Reading a message
async fn read_message(stream: &mut TcpStream) -> Result<Vec<u8>> {
    // Read length prefix
    let len = stream.read_u32().await? as usize;
    
    // Read exact number of bytes
    let mut buffer = vec![0u8; len];
    stream.read_exact(&mut buffer).await?;
    
    Ok(buffer)
}
```

**Key Decisions:**
- Use 4-byte length prefix (supports up to 4GB messages)
- Big-endian for network byte order
- read_exact ensures we get complete message

**What I Learned:**
- Stream protocols need explicit framing
- Length prefixing is simple and reliable
- tokio provides helpful read/write utilities
- Always consider partial reads/writes

**Alternative Approaches Considered:**
1. Newline delimited - Rejected: breaks with formatted JSON
2. Special delimiter sequence - Rejected: escaping complexity
3. Fixed-size messages - Rejected: inflexible
4. Length prefix - ✅ Chosen: simple, reliable, efficient

---

### Challenge 3: API Key Authentication
**Status:** ✅ Solved  
**Difficulty:** ⭐⭐⭐  

**The Problem:**
```
Need to authenticate clients to prevent unauthorized access to RPC methods.
Requirements:
- Simple to implement
- Secure enough for trusted networks
- Not too much overhead per request
- Easy to configure
```

**Approach:**
```rust
// Simple header-based authentication
struct ApiKeyAuth {
    valid_keys: HashSet<String>,
}

impl ApiKeyAuth {
    fn validate(&self, key: &str) -> bool {
        self.valid_keys.contains(key)
    }
}

// In HTTP handler
if let Some(api_key) = request.headers().get("X-API-Key") {
    if !auth.validate(api_key.to_str()?) {
        return Err(Error::Unauthorized);
    }
}
```

**Security Considerations:**
```
Current implementation:
- Plain text API keys (OK for trusted networks)
- Should use TLS in production
- Keys stored in config file

Future improvements:
- Hash API keys at rest
- Add rate limiting per key
- Key rotation mechanism
- JWT tokens for more features
```

**What I Learned:**
- Security is about threat models
- Simple auth is fine for trusted environments
- Always use TLS for production
- Think about key management early

---

### Challenge 4: Batch Request Processing
**Status:** ✅ Solved  
**Difficulty:** ⭐⭐⭐⭐  

**The Problem:**
```
JSON-RPC 2.0 spec allows batch requests (array of requests).
Each request should be processed independently, but responses
must be returned in a single batch response.

Challenges:
- Process requests concurrently for performance
- Maintain response ordering
- Handle mix of success/error results
- Don't block on slow requests
```

**Solution:**
```rust
async fn handle_batch(requests: Vec<Request>) -> Vec<Response> {
    // Spawn tasks for all requests concurrently
    let handles: Vec<_> = requests
        .into_iter()
        .enumerate()
        .map(|(idx, req)| {
            tokio::spawn(async move {
                let response = handle_single_request(req).await;
                (idx, response)
            })
        })
        .collect();
    
    // Collect results maintaining order
    let mut results = Vec::new();
    for handle in handles {
        let (idx, response) = handle.await.unwrap();
        results.push((idx, response));
    }
    
    // Sort by original index and extract responses
    results.sort_by_key(|(idx, _)| *idx);
    results.into_iter().map(|(_, resp)| resp).collect()
}
```

**What I Learned:**
- enumerate() helps maintain ordering
- tokio::spawn for concurrent execution
- Collect and sort pattern for ordered results
- Trade-offs between concurrency and complexity

---

### Challenge 5: Graceful Shutdown
**Status:** ✅ Solved  
**Difficulty:** ⭐⭐⭐  

**The Problem:**
```
When server receives shutdown signal (Ctrl+C), need to:
1. Stop accepting new connections
2. Wait for in-flight requests to complete
3. Close all connections cleanly
4. Exit without data loss

Without this, forceful shutdown could corrupt state or lose data.
```

**Solution:**
```rust
use tokio::sync::broadcast;
use tokio::signal;

async fn run_server() {
    let (shutdown_tx, _) = broadcast::channel(1);
    
    // Spawn shutdown listener
    let shutdown_listener = shutdown_tx.clone();
    tokio::spawn(async move {
        signal::ctrl_c().await.unwrap();
        println!("Shutdown signal received");
        shutdown_listener.send(()).unwrap();
    });
    
    // Main server loop
    loop {
        let mut shutdown_rx = shutdown_tx.subscribe();
        
        tokio::select! {
            result = listener.accept() => {
                // Handle new connection
            }
            _ = shutdown_rx.recv() => {
                println!("Shutting down gracefully...");
                break;
            }
        }
    }
    
    // Wait for tasks to complete
    tokio::time::sleep(Duration::from_secs(5)).await;
}
```

**What I Learned:**
- broadcast channel for shutdown signaling
- tokio::select! for multiple futures
- Give time for cleanup before exit
- Signal handling is platform-specific

---

## Ongoing Challenges

### Challenge 6: Error Handling Consistency
**Status:** 🔄 In Progress  
**Difficulty:** ⭐⭐⭐

**The Issue:**
```
Multiple error types across the codebase:
- IO errors
- Parse errors
- Handler errors
- Protocol errors

Need unified error handling strategy.
```

**Current Approach:**
```rust
// Using thiserror for custom error types
#[derive(Error, Debug)]
pub enum RpcError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("Parse error: {0}")]
    Parse(#[from] serde_json::Error),
    
    #[error("Method not found: {0}")]
    MethodNotFound(String),
}
```

**Next Steps:**
- [ ] Standardize error responses
- [ ] Better error messages
- [ ] Add error codes
- [ ] Context for debugging

---

### Challenge 7: Request Timeout Handling
**Status:** Planned  
**Difficulty:** ⭐⭐⭐

**The Issue:**
```
Long-running requests can hang indefinitely.
Need timeout mechanism to prevent resource exhaustion.
```

**Plan:**
```rust
// Wrap requests with timeout
use tokio::time::{timeout, Duration};

let result = timeout(
    Duration::from_secs(30),
    handle_request(request)
).await;
```

---

## Lessons Learned

### General Patterns
1. **Start simple, iterate** - Basic version first, optimize later
2. **Write tests early** - Catch issues before they compound
3. **Study existing code** - Learn from production implementations
4. **Document decisions** - Future you will thank you

### Rust-Specific
1. **Fight the borrow checker early** - Don't work around it
2. **Use Arc for shared state** - Clone Arc, not data
3. **Async is powerful but complex** - Understand task spawning
4. **Error handling with thiserror** - Makes custom errors easy

### Protocol Engineering
1. **Framing is crucial** - Don't overlook it
2. **Concurrency is hard** - But Rust helps
3. **Security from the start** - Retrofitting is harder
4. **Spec compliance matters** - Follow JSON-RPC 2.0 exactly

---

## Challenge Statistics

**Total Challenges:** 7  
**Solved:** 5  
**In Progress:** 1  
**Planned:** 1

**Average Time to Solve:** ~3-5 days per major challenge  
**Most Difficult:** Concurrent request handling (⭐⭐⭐⭐)  
**Most Valuable Learning:** Length-prefixed framing

---

## Future Challenges to Expect

### Performance
- [ ] Connection pooling
- [ ] Zero-copy serialization
- [ ] Request batching optimization

### Features
- [ ] WebSocket transport
- [ ] Compression support
- [ ] Metrics and monitoring

### Reliability
- [ ] Better error recovery
- [ ] Retry logic
- [ ] Circuit breakers

---
**Project Status:** Active Development