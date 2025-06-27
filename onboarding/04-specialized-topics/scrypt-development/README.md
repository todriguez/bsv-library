# sCrypt: Smart Contracts for BSV

## What is sCrypt?

sCrypt is a high-level programming language for writing smart contracts on BSV. Think of it as a bridge between the familiar world of TypeScript development and the powerful but low-level Bitcoin Script.

### Why Do We Need sCrypt?

Writing smart contracts directly in Bitcoin Script is like writing web applications in assembly language - it's possible, but extremely difficult and error-prone. sCrypt solves this by providing:

- **Type Safety**: Catch errors at compile time, not runtime
- **Familiar Syntax**: If you know TypeScript/JavaScript, you're already halfway there
- **High-Level Abstractions**: Work with concepts, not opcodes
- **Developer Tools**: Debugging, testing, and deployment frameworks

## Understanding sCrypt as an Embedded DSL

### What is an Embedded Domain Specific Language (eDSL)?

An embedded DSL is a specialized language that lives inside another language. In sCrypt's case:

```typescript
// This is regular TypeScript
class MyWebApp {
    private users: string[] = [];
    
    addUser(name: string) {
        this.users.push(name);
    }
}

// This is sCrypt (embedded in TypeScript)
class MySmartContract extends SmartContract {
    @prop()
    readonly answer: bigint;
    
    @method()
    public unlock(guess: bigint) {
        // This compiles to Bitcoin Script!
        assert(guess == this.answer, 'Wrong answer');
    }
}
```

### Key Insight: Not All TypeScript is sCrypt

**Important**: sCrypt is a *subset* of TypeScript. This means:
- ✅ All sCrypt code is valid TypeScript
- ❌ Not all TypeScript code is valid sCrypt

Think of it like this:
```
TypeScript (Full Language)
    └── sCrypt (Subset for Smart Contracts)
           └── Compiles to → Bitcoin Script
```

## How BSV Smart Contracts Work

### The UTXO Model Explained

Unlike account-based blockchains, BSV uses the UTXO (Unspent Transaction Output) model. Let's understand this step by step:

#### 1. What is a UTXO?

Imagine digital coins as locked boxes:
- Each box (UTXO) contains some BSV
- Each box has a lock (locking script)
- To spend the coins, you need the right key (unlocking script)

#### 2. Transaction Structure

A BSV transaction has:
```
INPUTS (References to existing UTXOs)
  ├── Previous Transaction ID
  └── Unlocking Script (the "key")

OUTPUTS (New UTXOs being created)
  ├── Amount of BSV
  └── Locking Script (the "lock")
```

#### 3. The Smart Contract Connection

Your sCrypt code becomes the locking script:

```typescript
// This sCrypt code...
@method()
public unlock(preimage: ByteString) {
    // Check if the provided data hashes to our target
    assert(sha256(preimage) == this.hash, 'Invalid preimage');
}

// ...compiles to Bitcoin Script that locks the coins
// Only someone with the correct preimage can unlock them!
```

### Conceptual Flow of a Smart Contract

```
DEPLOYMENT PHASE:
1. Write contract in sCrypt
2. Compile to Bitcoin Script
3. Create transaction with script as lock
4. Broadcast to BSV network

EXECUTION PHASE:
1. Someone wants to spend the locked coins
2. They provide unlocking data
3. BSV nodes run the script
4. If assertions pass → coins are spent ✓
5. If assertions fail → transaction rejected ✗
```

## Why TypeScript as the Host Language?

sCrypt chose TypeScript for excellent reasons:

### 1. **Type Safety**
```typescript
// TypeScript catches this error before deployment
@method()
public unlock(x: bigint) {
    // Error: Cannot compare bigint with string
    assert(x == "hello", 'Invalid');  // ❌ Compile error
}
```

### 2. **Familiar Syntax**
Millions of developers already know JavaScript/TypeScript, making sCrypt accessible.

### 3. **Modern Developer Experience**
- IntelliSense and auto-completion
- Integrated debugging
- Rich ecosystem of tools

### 4. **Progressive Learning**
Start with simple contracts and gradually adopt advanced features.

## sCrypt's Relationship to Bitcoin Script

### The Compilation Process

```
sCrypt Code (High Level)
    ↓ Compiler
Bitcoin Script (Low Level)
    ↓ Deployment
Locking Script in UTXO
```

### Example Transformation

```typescript
// sCrypt code (what you write)
@method()
public unlock(x: bigint) {
    assert(x == 42n, 'Not the answer');
}

// Compiles to Bitcoin Script (simplified)
// OP_42 OP_EQUAL OP_VERIFY
```

## Getting Started with sCrypt

### Prerequisites

Before diving into sCrypt:
1. Basic understanding of programming (any language)
2. Familiarity with BSV's UTXO model (covered above)
3. Node.js installed on your system

### Your First Steps

1. **[Installation](./installation.md)** - Set up your development environment
2. **[BSV Basics](./bsv-basics/README.md)** - Essential blockchain concepts
3. **[Writing Contracts](./writing-contracts/README.md)** - Create your first smart contract
4. **[Deployment](./deployment/README.md)** - Put contracts on the BSV network

## Key Concepts to Remember

### 1. **Contracts are Templates**
When you write a smart contract, you're creating a template. Each deployment creates a specific instance with its own parameters.

### 2. **Immutability**
Once deployed, contract code cannot be changed. The coins are locked by that specific script forever (until spent).

### 3. **Deterministic Execution**
Given the same inputs, a contract always produces the same result. No randomness or external API calls.

### 4. **Resource Constraints**
Contracts must be efficient. BSV has limits on script size and execution steps.

## Learning Path

### 🎯 Quick Start Track (1-2 hours)
1. Install sCrypt CLI
2. Create "Hello World" contract
3. Test locally
4. Deploy to testnet

### 📚 Comprehensive Track (1-2 weeks)
1. Master sCrypt syntax
2. Understand all decorators (@prop, @method)
3. Learn state management
4. Build real applications

### 🚀 Advanced Track (Ongoing)
1. Optimize gas costs
2. Complex contract patterns
3. Integration with LARS framework
4. Production deployment strategies

## Common Misconceptions

### ❌ "sCrypt is like Solidity"
No, sCrypt follows the UTXO model, not account-based. Contracts lock coins, not maintain global state.

### ❌ "I need to learn Bitcoin Script first"
No, sCrypt abstracts away the complexity. Start with sCrypt directly.

### ❌ "Smart contracts on BSV are limited"
No, BSV's restored opcodes and unlimited script size enable powerful contracts.

## Next Steps

Ready to start building? Here's your journey:

1. **[Installation Guide](./installation.md)** - Set up in 5 minutes
2. **[Tutorial: Hello World](./tutorials/hello-world.md)** - Your first contract
3. **[sCrypt by Example](./examples/README.md)** - Learn by doing

## Additional Resources

- **[LARS Framework](./deployment/lars-framework.md)** - Deploy contracts via configuration
- **[API Reference](./reference/README.md)** - Complete method documentation
- **[Best Practices](./best-practices.md)** - Production-ready patterns

---

> 💡 **Remember**: sCrypt makes BSV smart contract development accessible. You don't need to be a Bitcoin Script expert - just bring your programming skills and creativity!