# Writing Smart Contracts with sCrypt

## 🎯 Your Journey from TypeScript to Blockchain

Welcome to the exciting world of writing smart contracts! This section will teach you how to create powerful blockchain applications using sCrypt. We'll start simple and gradually build up to more complex concepts.

## 📚 What You'll Learn

By the end of this section, you'll understand:

1. **The sCrypt Programming Model** - How TypeScript becomes blockchain code
2. **Core Decorators** - The magic of `@prop`, `@method`, and `@state`
3. **Type System** - What types work on-chain and why
4. **Control Flow** - How assertions drive smart contract logic
5. **State Management** - Creating contracts that evolve over time
6. **Best Practices** - Writing secure, efficient contracts

## 🌟 Understanding the sCrypt Model

### The Two-World Paradigm

sCrypt contracts live in two worlds:

```typescript
// World 1: TypeScript (Development Time)
// This is where you write and test your code
class MyContract extends SmartContract {
    // Your TypeScript code here
}

// World 2: Bitcoin Script (Runtime)
// Your code compiles to this and runs on BSV
// OP_DUP OP_HASH160 ... (low-level opcodes)
```

### The Compilation Bridge

```
Your sCrypt Code (TypeScript)
         ↓
    sCrypt Compiler
         ↓
    Bitcoin Script
         ↓
  Deployed on BSV Blockchain
```

## 📖 Chapter Overview

### [1. Basics of Contract Writing](./basics.md)
Start here! Learn the fundamental concepts:
- Your first smart contract
- Understanding decorators (@prop, @method)
- The SmartContract base class
- Basic types and assertions

### [2. Built-in Functions](./built-ins.md)
Explore the powerful functions available:
- Cryptographic operations
- Mathematical functions
- Data manipulation
- Signature verification

### [3. Understanding ScriptContext](./scriptcontext.md)
Master transaction context:
- Accessing inputs and outputs
- Working with SigHash flags
- Building complex spending conditions
- UTXO state transitions

### [4. Stateful Contracts](./stateful-contracts.md)
Create contracts that maintain state:
- State vs stateless contracts
- The @state decorator
- State transitions
- Real-world examples

## 🚀 Quick Start: Your First Contract

Let's write a simple contract to get you started:

```typescript
import { SmartContract, method, prop, assert } from 'scrypt-ts'

// This contract locks funds until someone provides the correct answer
export class SimpleQuiz extends SmartContract {
    // @prop decorator: This value is stored on-chain when deployed
    // It becomes part of the locking script
    @prop()
    readonly answer: bigint

    // Constructor runs off-chain when creating the contract instance
    constructor(answer: bigint) {
        super(...arguments)
        this.answer = answer
    }

    // @method decorator: This becomes an on-chain function
    // It defines how the locked funds can be spent
    @method()
    public unlock(guess: bigint) {
        // assert is how we enforce rules on-chain
        // If this fails, the transaction is rejected
        assert(guess == this.answer, 'Wrong answer!')
    }
}
```

### What Just Happened?

1. **We defined a contract** that inherits from `SmartContract`
2. **We stored data** using `@prop` - this becomes part of the locking script
3. **We created a method** using `@method` - this defines spending conditions
4. **We used assert** to enforce our rule - correct answer unlocks funds

## 🎭 The Decorator Magic

### Understanding @prop

```typescript
@prop()
readonly x: bigint  // Stored on-chain, immutable

@prop(true)
count: bigint      // Stateful prop - can change between transactions
```

### Understanding @method

```typescript
@method()
public unlock() {   // Public method - entry point for spending
    // Your logic here
}

@method()
helper(): bigint {  // Non-public method - reusable logic
    return 42n
}
```

## 🔍 A Closer Look at Types

### Why Limited Types?

sCrypt only allows certain types because they must map to Bitcoin Script:

```typescript
// ✅ Allowed on-chain types
bigint          // Arbitrary precision integers
boolean         // true/false
ByteString      // Raw bytes
FixedArray<T>   // Arrays with compile-time known size

// ❌ Not allowed on-chain
number          // JavaScript numbers aren't precise enough
string          // Use ByteString instead
Array<T>        // Dynamic arrays aren't supported
objects         // No complex objects on-chain
```

### Type Conversion Examples

```typescript
// Converting between types
const num: bigint = 100n                          // bigint literal
const bytes: ByteString = toByteString('hello')   // string to bytes
const hex: ByteString = toByteString('deadbeef')  // hex to bytes
const bool: boolean = true                        // boolean
```

## 💡 Key Concepts to Master

### 1. **Assertions are Control Flow**

In traditional programming:
```typescript
// Traditional: return values
function validate(x: number): boolean {
    return x > 0
}
```

In sCrypt:
```typescript
// sCrypt: assertions
@method()
public validate(x: bigint) {
    assert(x > 0n, 'Must be positive')
    // No return needed - assertion is the control
}
```

### 2. **Contracts are Templates**

```typescript
// This is a template
class HashLock extends SmartContract {
    @prop()
    readonly hash: ByteString
    // ...
}

// Each deployment creates an instance with specific values
const contract1 = new HashLock(hash1)  // Instance 1
const contract2 = new HashLock(hash2)  // Instance 2
```

### 3. **On-chain vs Off-chain**

```typescript
export class MyContract extends SmartContract {
    // On-chain: compiled to Bitcoin Script
    @method()
    public unlock() {
        assert(true, 'Always unlockable')
    }
    
    // Off-chain: TypeScript helper (not compiled)
    async deploy() {
        // Deployment logic runs in Node.js
    }
}
```

## 🏗️ Building Your First Real Contract

Let's build something more practical - a simple escrow:

```typescript
export class Escrow extends SmartContract {
    // Three parties involved
    @prop()
    readonly buyer: PubKey
    
    @prop()
    readonly seller: PubKey
    
    @prop()
    readonly arbiter: PubKey
    
    @method()
    public release(buyerSig: Sig, arbiterSig: Sig) {
        // Buyer and arbiter agree to release funds
        assert(this.checkSig(buyerSig, this.buyer), 'Invalid buyer signature')
        assert(this.checkSig(arbiterSig, this.arbiter), 'Invalid arbiter signature')
    }
    
    @method()
    public refund(sellerSig: Sig, arbiterSig: Sig) {
        // Seller and arbiter agree to refund
        assert(this.checkSig(sellerSig, this.seller), 'Invalid seller signature')
        assert(this.checkSig(arbiterSig, this.arbiter), 'Invalid arbiter signature')
    }
}
```

## 📝 Best Practices

### 1. **Keep It Simple**
Start with simple contracts and gradually add complexity.

### 2. **Test Thoroughly**
Every assertion is a potential failure point - test all paths.

### 3. **Comment Your Code**
Explain the "why" not just the "what":

```typescript
// Good: Explains purpose
// Require 2-of-3 signatures to prevent single point of failure
assert(validSigs >= 2n, 'Insufficient signatures')

// Bad: States the obvious
// Check if validSigs is at least 2
assert(validSigs >= 2n, 'Insufficient signatures')
```

### 4. **Validate Everything**
Never trust external input:

```typescript
@method()
public unlock(x: bigint, y: bigint) {
    // Validate ranges
    assert(x > 0n && x < 1000n, 'X out of range')
    assert(y > 0n && y < 1000n, 'Y out of range')
    
    // Then use values
    assert(x + y == this.sum, 'Invalid sum')
}
```

## 🎓 Learning Path

### Beginner (Start Here)
1. Read [Basics](./basics.md) thoroughly
2. Try modifying the simple examples
3. Write your own simple contracts

### Intermediate
1. Study [Built-in Functions](./built-ins.md)
2. Experiment with different functions
3. Build more complex logic

### Advanced
1. Master [ScriptContext](./scriptcontext.md)
2. Create [Stateful Contracts](./stateful-contracts.md)
3. Build production-ready applications

## 🚦 Ready to Start?

Begin with [Contract Basics](./basics.md) to start your journey into smart contract development!

Remember:
- **Start simple** - Master basics before moving to complex contracts
- **Experiment freely** - Use testnet for learning
- **Ask questions** - The community is here to help

---

> 💡 **Pro Tip**: The best way to learn is by doing. After reading each section, try writing your own variations of the example contracts!