# Development Tools & Resources

## Tools by Category

Track the tools you use and want to learn for protocol engineering.

---

## 🦀 Rust Development

### Core Tools
- [ ] **rustc** - Rust compiler
  - Status: Using | Learning | Haven't tried
  - Notes: 

- [ ] **cargo** - Package manager and build tool
  - Status: Using daily
  - Useful commands:
    ```bash
    cargo new, cargo build, cargo test, cargo bench
    ```

- [ ] **rustfmt** - Code formatter
  - Status: Using
  - Config: 

- [ ] **clippy** - Linter
  - Status: Using
  - Notes: 

### Async Runtime
- [ ] **tokio** - Async runtime
  - Status: Using (DiceRPC)
  - Notes: Main async runtime, great docs

- [ ] **async-std** - Alternative async runtime
  - Status: Haven't tried
  - Notes: 

### Testing
- [ ] **cargo test** - Built-in testing
  - Status: Using

- [ ] **proptest** - Property-based testing
  - Status: Want to learn

- [ ] **criterion** - Benchmarking
  - Status: Want to learn

### Debugging & Profiling
- [ ] **gdb** / **lldb** - Debuggers
  - Status: Basic usage

- [ ] **cargo flamegraph** - Profiling
  - Status: Haven't tried

- [ ] **perf** - Performance analysis
  - Status: Want to learn

---

## Networking Libraries

### P2P Networking
- [ ] **libp2p** - Modular P2P networking
  - Status: Learning
  - Notes: Used in many protocols

- [ ] **tokio** - Async TCP/UDP
  - Status: Using

### RPC/API
- [ ] **tonic** - gRPC for Rust
  - Status: Want to learn

- [ ] **jsonrpc** - JSON-RPC implementations
  - Status: Built my own (DiceRPC)

- [ ] **axum** - Web framework
  - Status: Haven't tried

- [ ] **actix-web** - Web framework
  - Status: Haven't tried

---

## Cryptography Libraries

### Hashing
- [ ] **sha2** - SHA-256, SHA-512
  - Status: Basic usage

- [ ] **sha3** - Keccak
  - Status: Want to use

- [ ] **blake3** - BLAKE3 hashing
  - Status: Haven't tried

### Signatures
- [ ] **secp256k1** - Bitcoin signatures
  - Status: Basic usage

- [ ] **ed25519-dalek** - EdDSA signatures
  - Status: Want to learn

- [ ] **bls-signatures** - BLS signatures
  - Status: Haven't tried

### Encryption
- [ ] **aes-gcm** - AES encryption
  - Status: Basic knowledge

- [ ] **ring** - Cryptography library
  - Status: Want to learn

### Zero-Knowledge
- [ ] **arkworks** - ZK proof framework
  - Status: Want to learn (advanced)

---

## Storage & Databases

### Embedded Databases
- [ ] **sled** - Embedded DB in Rust
  - Status: Want to try

- [ ] **RocksDB** - Used in blockchains
  - Status: Basic understanding

- [ ] **LevelDB** - Used in Bitcoin
  - Status: Basic understanding

### Serialization
- [ ] **serde** - Serialization framework
  - Status: Using

- [ ] **bincode** - Binary serialization
  - Status: Basic usage

- [ ] **protobuf** - Protocol Buffers
  - Status: Want to learn

---

## Testing & Simulation

### Testing Frameworks
- [ ] **cargo test** - Built-in
  - Status: Using

- [ ] **rstest** - Fixture-based testing
  - Status: Want to try

- [ ] **proptest** - Property testing
  - Status: Want to learn

### Mocking
- [ ] **mockall** - Mock objects
  - Status: Want to learn

### Fuzzing
- [ ] **cargo-fuzz** - Fuzzing tool
  - Status: Want to learn

- [ ] **honggfuzz** - Fuzzer
  - Status: Haven't tried

---

## Analysis Tools

### Code Analysis
- [ ] **cargo-audit** - Security audit
  - Status: Should use

- [ ] **cargo-deny** - Lint dependencies
  - Status: Want to setup

- [ ] **cargo-geiger** - Unsafe code detection
  - Status: Want to try

### Performance
- [ ] **cargo bench** - Benchmarking
  - Status: Basic usage

- [ ] **criterion** - Statistics-driven benchmarking
  - Status: Want to learn

- [ ] **flamegraph** - Performance profiling
  - Status: Want to learn

---

## Blockchain Development

### Ethereum Development
- [ ] **Foundry** - Rust-based toolkit
  - Status: Want to learn
  - Components: forge, cast, anvil

- [ ] **Hardhat** - TypeScript toolkit
  - Status: Basic usage (Own Protocol)

- [ ] **Remix** - Web IDE
  - Status: Basic usage

### Web3 Libraries (Rust)
- [ ] **ethers-rs** - Ethereum library
  - Status: Want to learn

- [ ] **alloy** - Next-gen Ethereum library
  - Status: Haven't tried

### Web3 Libraries (TypeScript)
- [ ] **ethers.js** - Ethereum library
  - Status: Using (Own Protocol work)

- [ ] **viem** - Modern web3 library
  - Status: Using (Own Protocol work)

- [ ] **web3.js** - Original web3 library
  - Status: Basic familiarity

---

## Node & Client Software

### Ethereum Clients
- [ ] **Geth** (Go)
  - Status: Basic understanding

- [ ] **Reth** (Rust)
  - Status: Want to study

- [ ] **Lighthouse** (Rust consensus)
  - Status: Want to study

### Bitcoin Clients
- [ ] **Bitcoin Core** (C++)
  - Status: Basic understanding

---

## Monitoring & Observability

### Logging
- [ ] **tracing** - Structured logging
  - Status: Want to use

- [ ] **log** - Logging facade
  - Status: Basic usage

### Metrics
- [ ] **prometheus** - Metrics
  - Status: Want to learn

### Distributed Tracing
- [ ] **jaeger** - Tracing
  - Status: Want to learn

---

## DevOps & Infrastructure

### Containerization
- [ ] **Docker**
  - Status: Basic usage

- [ ] **Kubernetes**
  - Status: Want to learn basics

### CI/CD
- [ ] **GitHub Actions**
  - Status: Use ut everyday

---

## Documentation

### Rust Docs
- [ ] **rustdoc** - Generate docs
  - Status: Using

- [ ] **mdBook** - Create books
  - Status: Haven't tried
### Writing for research, personal learnings and findings

- [ ] **hackMD** - Generate docs, Articles and research papers
  - Status: Using

---

## Learning Resources

### Interactive Tools
- [ ] **Rust Playground** - Online Rust
  - https://play.rust-lang.org

- [ ] **Remix** - Solidity IDE
  - https://remix.ethereum.org

### Visualization
- [ ] **excalidraw** - Diagrams
  - Status: Want to use for architecture

---

## Tool Learning Goals

### This Month
- [ ] Master [tool]
- [ ] Try [tool]
- [ ] Setup [tool] in my workflow

### This Quarter
- [ ] [Goal]
- [ ] [Goal]

---

## Tool Notes Template

When you learn a new tool:

```markdown
### [Tool Name]
**Category:** [Category]  
**Language:** [Language if applicable]  
**Status:** [ ] Want to learn | [ ] Learning | [ ] Using | [ ] Proficient

**Installation:**
```bash
[How to install]
```

**Basic Usage:**
```bash
[Common commands/patterns]
```

**When to Use:**
[Use cases]

**Pros:**
- 
- 

**Cons:**
- 
- 

**Resources:**
- [Official docs]
- [Tutorial]

**My Notes:**
```
[Your experience using it]

Tips:
- 

Gotchas:
- 
```

**Related Tools:**
- [Alternative 1]
- [Complement tool]

**Projects Using This:**
- [Your project]
```
---