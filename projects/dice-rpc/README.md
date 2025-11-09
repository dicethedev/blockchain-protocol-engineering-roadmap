# DiceRPC - JSON-RPC 2.0 Framework

## Project Overview

**Status:** Phase 1 Complete | Active Development  
**Repository:** [DiceRPC](https://github.com/dicethedev/DiceRPC)
**HackMD Article:** [Building-a-Lightning-Fast-JSON-RPC-Server-in-Rust-🦀]https://hackmd.io/AJz1P0gISx6W0TEewLRJ3w
**Language:** Rust  
**Current Version:** v0.1.0

### Description
A production-ready JSON-RPC 2.0 framework built from scratch in Rust, demonstrating deep understanding of async programming, network protocols, and concurrent systems design.

### Key Features Implemented
- ✅ **JSON-RPC 2.0 Spec Compliance** - Full implementation of the specification
- ✅ **Dual Transports** - TCP with length-prefixed framing + HTTP
- ✅ **Concurrent Request Handling** - Powered by tokio async runtime
- ✅ **API Key Authentication** - Simple but effective access control
- ✅ **Batch Request Support** - Handle multiple requests efficiently
- ✅ **Graceful Shutdown** - Clean resource cleanup on exit
- ✅ **Type-Safe Error Handling** - Custom error types with rich context

### Project Goals
This project was built to:
1. Master async Rust programming with tokio
2. Understand network protocol design and implementation
3. Learn concurrent systems programming patterns
4. Build a real-world, spec-compliant protocol implementation
5. Develop foundation for blockchain protocol engineering

---

## Documentation

This project is fully documented across multiple files:

- **[learnings.md](./learnings.md)** - Deep dive into everything I learned (Rust, networking, protocols)
- **[challenges.md](./challenges.md)** - Technical challenges faced and solutions
- **README.md** (this file) - Quick overview and architecture

**Start here:** Read [learnings.md](./learnings.md) for the full story of building this project.

---

## Architecture

### High-Level Design

``mermaid
graph TB
    subgraph Client["Client Layer"]
        CLI[CLI Client]
        EXT[External Clients]
    end

    subgraph Transport["Transport Layer"]
        TCP[TCP Transport]
        HTTP[HTTP Transport]
    end

    subgraph Middleware["Middleware Layer"]
        AUTH[Authentication]
        BATCH[Batch Handler]
    end

    subgraph Core["Core RPC Engine"]
        ROUTER[Request Router]
        REGISTRY[Handler Registry]
        EXEC[Async Executor]
    end

    subgraph Logic["Business Logic"]
        H1[ping]
        H2[get_balance]
        H3[transfer]
        H4[get_transaction]
    end

    subgraph State["State Management"]
        STORE[StateStore]
        MEM[In-Memory DB]
    end

    subgraph Observability["Observability"]
        METRICS[Metrics]
        LOGS[Logging]
    end

    CLI --> TCP
    EXT --> HTTP
    EXT --> TCP

    TCP --> BATCH
    HTTP --> BATCH

    BATCH --> AUTH
    AUTH --> ROUTER
    ROUTER --> REGISTRY
    REGISTRY --> EXEC

    EXEC --> H1
    EXEC --> H2
    EXEC --> H3
    EXEC --> H4

    H2 --> STORE
    H3 --> STORE
    H4 --> STORE

    STORE --> MEM

    ROUTER --> METRICS
    AUTH --> LOGS

    style TCP fill:#e1f5ff
    style HTTP fill:#e1f5ff
    style AUTH fill:#fff3e0
    style ROUTER fill:#f3e5f5
    style STORE fill:#e8f5e9
    style METRICS fill:#fce4ec
```

### Key Components

**Transport Abstraction:**
- TCP with 4-byte length-prefix framing
- HTTP with standard request/response
- Shared handler interface

**Concurrency Model:**
- Task-per-connection using tokio::spawn
- Arc<Mutex<T>> for shared state
- Channel-based shutdown coordination

**Error Handling:**
- Custom error types with thiserror
- JSON-RPC 2.0 compliant error codes
- Rich error context for debugging

**See [learnings.md](./learnings.md)** for deep dive into architecture decisions

---

## Major Challenges Solved

I encountered and solved several complex challenges:

1. **Concurrent Request Handling** - tokio tasks + Arc pattern
2. **Length-Prefixed Framing** - Reliable message boundaries over TCP
3. **Batch Processing** - Concurrent execution with ordered responses
4. **Graceful Shutdown** - broadcast channel coordination
5. **API Key Authentication** - Header-based auth with validation

**See [challenges.md](./challenges.md)** for detailed solutions and learnings

---

## What I Learned

### Technical Skills Gained
- ✅ **Async Rust Mastery** - tokio runtime, futures, task spawning
- ✅ **Network Programming** - TCP streams, framing, transport design
- ✅ **Concurrent Systems** - Safe shared state, race condition prevention
- ✅ **Protocol Implementation** - JSON-RPC 2.0 spec compliance
- ✅ **Error Handling** - Type-safe errors with context

### Protocol Engineering Insights
- ✅ Stream protocols need explicit framing (length-prefix pattern)
- ✅ Spec compliance is critical for interoperability
- ✅ Concurrency is hard but Rust makes it safe
- ✅ API design matters as much as implementation
- ✅ Testing and observability from day one

**See [learnings.md](./learnings.md)** for complete learning breakdown

---

## Code Highlights

### Pattern 1: Concurrent Request Handling
```rust
// Each connection runs in its own task
async fn accept_connections(listener: TcpListener, handler: Arc<dyn RpcHandler>) {
    loop {
        let (stream, _addr) = listener.accept().await?;
        let handler = handler.clone(); // Clone Arc, not data
        
        tokio::spawn(async move {
            handle_connection(stream, handler).await
        });
    }
}
```
**Why it's powerful:** Thousands of clients can connect without blocking each other

### Pattern 2: Length-Prefixed Framing
```rust
// TCP is a stream - we need message boundaries
async fn write_message(stream: &mut TcpStream, data: &[u8]) -> Result<()> {
    // 4-byte length prefix (big-endian)
    stream.write_u32(data.len() as u32).await?;
    stream.write_all(data).await?;
    Ok(())
}
```
**Why it matters:** Solves the fundamental problem of delimiting messages over streams

### Pattern 3: Type-Safe Error Handling
```rust
#[derive(Error, Debug)]
pub enum RpcError {
    #[error("Network error: {0}")]
    Network(#[from] std::io::Error),
    
    #[error("Method not found: {0}")]
    MethodNotFound(String),
}
```
**Why it's elegant:** Rust's type system prevents error handling bugs at compile time

---

## Performance

### Benchmarks (on my machine)
```
Sequential baseline: ~100 req/sec
Concurrent (4 cores): ~5,000 req/sec
Optimized: ~8,000 req/sec

Latency:
- p50: 0.5ms
- p95: 2ms
- p99: 5ms
```

### Optimizations Applied
- Task spawning for concurrency
- Arc for zero-copy state sharing
- Minimal lock contention
- Efficient async I/O with tokio

---

## Next Steps

### Phase 1: Core Improvements (Next Month)
- [ ] Request timeout handling
- [ ] Enhanced error messages with context
- [ ] Comprehensive logging with tracing
- [ ] >85% test coverage

### Phase 2: Feature Additions (1-2 Months)
- [ ] WebSocket transport
- [ ] Request/response compression
- [ ] Prometheus metrics
- [ ] Rate limiting
- [ ] TLS support

### Phase 3: Production Hardening
- [ ] Connection pooling
- [ ] Circuit breaker pattern
- [ ] Zero-copy optimizations
- [ ] Complete documentation
---

## Testing

### Current Coverage
- Unit tests: Core logic and framing
- Integration tests: End-to-end request flow
- Manual testing: Multiple clients, edge cases

### Testing Challenges
- Async testing with tokio::test
- Concurrent scenarios
- Network I/O mocking
- Race condition detection

**Goal:** >85% code coverage with comprehensive test suite

---

## Resources Used

### Essential Reading
- ✅ JSON-RPC 2.0 Specification
- ✅ Tokio async book
- ✅ Rust async book
- ✅ thiserror documentation

### Code Studied
- jsonrpc-core crate (reference implementation)
- tonic (gRPC) for transport patterns
- Production RPC servers

### Tools
- Rust analyzer for IDE support
- cargo clippy for lints
- tokio-console for async debugging

---

## 🎓 Impact on My Learning Journey

### Skills Progression
| Skill | Before | After | Growth |
|-------|--------|-------|--------|
| Rust | ⭐⭐ | ⭐⭐⭐⭐ | +2 levels |
| Async Programming | ⭐ | ⭐⭐⭐⭐ | +3 levels |
| Networking | ⭐⭐ | ⭐⭐⭐⭐ | +2 levels |
| Protocol Design | ⭐ | ⭐⭐⭐ | +2 levels |
| Concurrent Systems | ⭐⭐ | ⭐⭐⭐⭐ | +2 levels |

### Transferable Learnings
This project gave me skills directly applicable to:
- Blockchain protocol implementation
- P2P networking systems
- High-performance backend services
- Distributed systems engineering

### Confidence Gained
**Before:** Could write basic Rust, unsure about async/networking  
**After:** Can build production-quality networked systems from scratch

---

## Key Achievements

### Technical
- ✅ Built spec-compliant JSON-RPC 2.0 framework
- ✅ Mastered async Rust with tokio
- ✅ Solved complex concurrency challenges
- ✅ Implemented reliable network protocol

### Learning
- ✅ Deep understanding of stream protocols
- ✅ Practical experience with Rust ownership
- ✅ Real-world protocol engineering
- ✅ Production code quality awareness

### Portfolio
- ✅ Demonstrable systems programming project
- ✅ Shows protocol engineering capability
- ✅ Proves Rust proficiency
- ✅ Foundation for future projects

---

## What I'd Do Differently

### If Starting Over
1. **Write tests first** - TDD would have caught bugs earlier
2. **Design API thoroughly** - Better interface from day one
3. **Start with HTTP** - Add TCP complexity later
4. **Logging from start** - Would have simplified debugging

### What Went Right
1. ✅ Incremental development approach
2. ✅ Following spec strictly
3. ✅ Clean layered architecture
4. ✅ Good documentation habits

---

## Related Blog Posts (Ideas)

- [ ] "Building a JSON-RPC Framework in Rust: Lessons Learned"
- [ ] "Async Rust in Practice: From Zero to RPC Server"
- [ ] "The Hidden Complexity of Stream Protocols"
- [ ] "Why Rust's Type System Makes Network Programming Safer"
- [ ] "Concurrency Without Fear: tokio and Arc"

---

## 🔗 Quick Links

- [Detailed Learnings](./learnings.md) - Full breakdown of what I learned
- [Challenges Solved](./challenges.md) - Technical problems and solutions
- Main Roadmap: [Protocol Design](../../roadmap/05-protocol-design/rpc-systems.md)
- Code Patterns: [Rust Async Patterns](../../code-patterns/rust/README.md)

---

**Project Status:** Phase 1 Complete ✅  
**Current Focus:** Planning Phase 2 features  

**Next Project:** [p2p-chat](../p2p-chat/README.md)