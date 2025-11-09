# Important Repositories to Study

## Overview
A curated list of codebases to learn from. Study these to understand production-quality protocol engineering.

---

## Essential Codebases

### 1. Bitcoin Core
**Language:** C++  
**Repository:** https://github.com/bitcoin/bitcoin  
**Status:** [ ] Haven't looked | [ ] Browsing | [ ] Studying | [ ] Understand well

**Why Study This:**
```
- Original blockchain implementation
- Production-grade P2P networking
- Consensus implementation
- UTXO model
```

**Areas to Focus:**
- [ ] P2P networking code
- [ ] Consensus validation
- [ ] Transaction processing
- [ ] Block propagation

**My Notes:**
```
[What you learned]

Interesting patterns:
- 

Code quality insights:
- 
```

**Files/Modules to Study:**
```
- src/net.cpp - Networking
- src/validation.cpp - Consensus
- src/txmempool.cpp - Mempool
```

---

### 2. Ethereum (Geth)
**Language:** Go  
**Repository:** https://github.com/ethereum/go-ethereum  
**Status:** [ ] Haven't looked | [ ] Browsing | [ ] Studying | [ ] Understand well

**Why Study This:**
```
- EVM implementation
- Account model
- devp2p protocol
- State management with MPT
```

**Areas to Focus:**
- [ ] EVM implementation
- [ ] State management
- [ ] P2P networking (devp2p)
- [ ] Transaction pool

**My Notes:**
```
[Your notes]
```

**Files/Modules to Study:**
```
- core/vm/ - EVM
- core/state/ - State management
- p2p/ - Networking
```

---

### 3. Reth (Rust Ethereum)
**Language:** Rust  
**Repository:** https://github.com/paradigmxyz/reth  
**Status:** [ ] Haven't looked | [ ] Browsing | [ ] Studying | [ ] Understand well

**Why Study This:**
```
- Modern Rust implementation
- Excellent code quality
- Performance optimizations
- Modular architecture
```

**My Notes:**
```
[Your notes]
```

---

## Important Protocol Implementations

### Consensus Implementations

#### Tendermint Core
**Language:** Go  
**Repository:** https://github.com/tendermint/tendermint  
**Focus:** BFT consensus  
**Status:** [ ] Not studied  
**Notes:**
```
[Your notes]
```

#### Lighthouse (Ethereum Consensus Client)
**Language:** Rust  
**Repository:** https://github.com/sigp/lighthouse  
**Focus:** Ethereum PoS  
**Status:** [ ] Not studied  
**Notes:**

---

### P2P Networking

#### rust-libp2p
**Language:** Rust  
**Repository:** https://github.com/libp2p/rust-libp2p  
**Focus:** Modular P2P networking  
**Status:** [ ] Not studied  
**Notes:**
```
[Your notes on libp2p]
```

#### Gossipsub
**Focus:** Pubsub over libp2p  
**Status:** [ ] Not studied  
**Notes:**

---

### Layer 2 Solutions

#### Optimism
**Repository:** https://github.com/ethereum-optimism/optimism  
**Focus:** Optimistic rollup  
**Status:** [ ] Not studied  
**Notes:**

#### Arbitrum
**Focus:** Optimistic rollup  
**Status:** [ ] Not studied  
**Notes:**

---

## Specialized Implementations

### Smart Contract Platforms

#### Solana
**Language:** Rust  
**Repository:** https://github.com/solana-labs/solana  
**Focus:** High-performance blockchain  
**Status:** [ ] Not studied  
**Notes:**

#### NEAR Protocol
**Language:** Rust  
**Focus:** Sharded blockchain  
**Status:** [ ] Not studied  
**Notes:**

---

### DeFi Protocols

#### Uniswap V3
**Language:** Solidity  
**Focus:** AMM with concentrated liquidity  
**Status:** [ ] Not studied  
**Notes:**

#### Compound
**Language:** Solidity  
**Focus:** Lending protocol  
**Status:** [ ] Not studied  
**Notes:**

---

### Cryptography Libraries

#### arkworks
**Language:** Rust  
**Focus:** Zero-knowledge proofs  
**Repository:** https://github.com/arkworks-rs  
**Status:** [ ] Not studied  
**Notes:**

#### libsecp256k1
**Language:** C  
**Focus:** ECDSA for Bitcoin  
**Status:** [ ] Not studied  
**Notes:**

---

### Development Tools

#### Foundry
**Language:** Rust  
**Focus:** Ethereum development toolkit  
**Repository:** https://github.com/foundry-rs/foundry  
**Status:** [ ] Not studied  
**Notes:**

#### Hardhat
**Language:** TypeScript  
**Focus:** Ethereum development environment  
**Status:** [ ] Not studied  
**Notes:**

---

## Repository Study Template

When studying a new codebase:

```markdown
### [Project Name]
**Language:** [Primary language]  
**Repository:** [GitHub URL]  
**Stars:** [Count]  
**Status:** [ ] Not studied | [ ] Browsing | [ ] Deep dive | [ ] Understand well

**Date Started:** [Date]  
**Time Invested:** [Hours]

**Why I'm Studying This:**
[Your reason]

**Architecture Overview:**
```
[High-level architecture notes]

Main components:
1. 
2. 
3. 
```

**Key Files/Modules:**
- `[path/file]` - [What it does]
- `[path/file]` - [What it does]

**Interesting Patterns Discovered:**
1. 
2. 
3. 

**Code Quality Observations:**
```
[Your thoughts on code quality]

Good practices:
- 

Could improve:
- 
```

**What I Learned:**
1. 
2. 
3. 

**Questions:**
1. 
2. 

**How This Helps My Projects:**
[Applicable patterns/ideas]

**Rating:** ⭐⭐⭐⭐⭐ (1-5 stars)
```

---

## Study Goals

### This Month
- [ ] Deep dive into [repository]
- [ ] Understand [specific component]

### This Quarter
- [ ] Study [X] repositories
- [ ] Focus on [area]

---

## Tips for Studying Code

1. **Start with README:** Understand the project goals
2. **Architecture First:** Get high-level view before diving into details
3. **Follow the Flow:** Trace a request/transaction through the system
4. **Read Tests:** Tests show how components are used
5. **Focus Areas:** Don't try to understand everything at once
6. **Take Notes:** Document patterns and learnings
7. **Ask Why:** Understand design decisions
8. **Compare:** Compare implementations across projects

---

## Study Stats

**Repositories Studied:** X  
**Currently Studying:** X  
**On Watchlist:** X  
**Average Study Time per Repo:** X hours