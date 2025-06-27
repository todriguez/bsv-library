# Contract Basics

## 🎯 Building Your Foundation

This chapter covers the fundamental concepts of writing sCrypt smart contracts. By the end, you'll understand how to create, structure, and compile your own contracts.

## 📝 Basic Contract Structure

Every sCrypt smart contract follows this pattern:

```typescript
// 1. Import necessary components from scrypt-ts
import { SmartContract, method, prop, assert } from "scrypt-ts"

// 2. Define your contract class extending SmartContract
export class MyContract extends SmartContract {
    // 3. Declare on-chain properties with @prop
    @prop()
    readonly myProperty: bigint
    
    // 4. Initialize via constructor
    constructor(value: bigint) {
        super(...arguments)  // Always call super first!
        this.myProperty = value
    }
    
    // 5. Define on-chain methods with @method
    @method()
    public unlock(solution: bigint) {
        // 6. Use assertions to enforce rules
        assert(solution == this.myProperty, 'Wrong solution!')
    }
}
```

## 🔑 The SmartContract Base Class

All contracts inherit from `SmartContract`:

```typescript
// SmartContract provides the foundation for all contracts
// It handles compilation, deployment, and interaction
export class YourContract extends SmartContract {
    // Your contract logic here
}
```

### What SmartContract Provides:

- **Compilation infrastructure** - Transforms your TypeScript to Bitcoin Script
- **Deployment helpers** - Methods to deploy to BSV network
- **State management** - For stateful contracts
- **Type checking** - Ensures on-chain type safety

## 🏷️ The @prop Decorator

The `@prop` decorator marks properties that become part of your contract's on-chain data:

```typescript
export class PropertyExamples extends SmartContract {
    // Immutable property - set once during deployment
    @prop()
    readonly fixedValue: bigint
    
    // Stateful property - can change between transactions
    @prop(true)
    counter: bigint
    
    // Boolean property
    @prop()
    readonly isActive: boolean
    
    // ByteString for arbitrary data
    @prop()
    readonly dataHash: ByteString
    
    // Fixed-size array
    @prop()
    readonly numbers: FixedArray<bigint, 3>
    
    constructor(value: bigint, active: boolean, hash: ByteString) {
        super(...arguments)
        this.fixedValue = value
        this.counter = 0n  // Initialize stateful property
        this.isActive = active
        this.dataHash = hash
        this.numbers = [1n, 2n, 3n]  // Must match declared size
    }
}
```

### Key Points About @prop:

1. **Immutable by default** - Use `readonly` for values that never change
2. **Stateful with @prop(true)** - Allows updates between transactions
3. **Limited types** - Only on-chain compatible types allowed
4. **Serialized to script** - Becomes part of the locking script

## 🎯 The @method Decorator

Methods decorated with `@method` compile to Bitcoin Script:

```typescript
export class MethodExamples extends SmartContract {
    @prop()
    readonly target: bigint
    
    constructor(target: bigint) {
        super(...arguments)
        this.target = target
    }
    
    // Public method - entry point for unlocking
    @method()
    public unlock(guess: bigint) {
        // This entire method compiles to Bitcoin Script
        assert(guess == this.target, 'Wrong guess!')
    }
    
    // Public method with multiple parameters
    @method()
    public unlockWithSum(a: bigint, b: bigint) {
        assert(a + b == this.target, 'Sum does not match target')
    }
    
    // Non-public method - internal helper
    @method()
    validate(value: bigint): boolean {
        // Can be called by other @method functions
        return value > 0n && value < 100n
    }
    
    // Public method using helper
    @method()
    public unlockInRange(value: bigint) {
        // First validate range
        assert(this.validate(value), 'Value out of range')
        // Then check against target
        assert(value == this.target, 'Wrong value')
    }
}
```

### Method Rules:

1. **Public methods** - Entry points callable from transactions
2. **At least one required** - Every contract needs ≥1 public method
3. **No return for public** - Public methods must return void
4. **Helpers allowed** - Non-public methods can return values

## 📊 Allowed Data Types

sCrypt supports specific types that map to Bitcoin Script:

```typescript
export class TypeExamples extends SmartContract {
    // Numeric type - arbitrary precision integers
    @prop()
    readonly amount: bigint  // Use 'n' suffix: 100n
    
    // Boolean type
    @prop()
    readonly isEnabled: boolean
    
    // Byte string - raw bytes or hex data
    @prop()
    readonly hash: ByteString
    
    // Fixed-size array (size must be known at compile time)
    @prop()
    readonly fixedList: FixedArray<bigint, 5>
    
    // Signatures and public keys
    @prop()
    readonly owner: PubKey
    
    // Hashed public key
    @prop()
    readonly recipient: PubKeyHash
    
    constructor() {
        super(...arguments)
        
        // Initialize with proper types
        this.amount = 1000n  // Note the 'n' suffix for bigint
        this.isEnabled = true
        this.hash = toByteString('0x1234abcd')  // Hex string to ByteString
        this.fixedList = [1n, 2n, 3n, 4n, 5n]  // Exactly 5 elements
        this.owner = PubKey(toByteString('0x02...'))  // 33-byte compressed pubkey
        this.recipient = PubKeyHash(toByteString('0x...'))  // 20-byte hash
    }
    
    @method()
    public demonstrateTypes(
        num: bigint,
        flag: boolean,
        data: ByteString,
        pubkey: PubKey,
        sig: Sig
    ) {
        // Working with different types
        assert(num > 0n, 'Must be positive')
        assert(flag == true, 'Flag must be true')
        assert(len(data) == 32n, 'Data must be 32 bytes')
        assert(this.checkSig(sig, pubkey), 'Invalid signature')
    }
}
```

### Type Conversion Utilities:

```typescript
// String to ByteString
const bytes1 = toByteString('Hello')  // UTF-8 encoding
const bytes2 = toByteString('0xdeadbeef')  // Hex encoding

// Number to bigint
const big1 = BigInt(42)  // From number
const big2 = 42n  // Literal syntax (preferred)

// Boolean conversions
const bool1 = true
const bool2 = (1n == 1n)  // From comparison
```

## ✅ The Assertion Pattern

Assertions are the primary control flow mechanism in sCrypt:

```typescript
export class AssertionExamples extends SmartContract {
    @prop()
    readonly target: bigint
    
    constructor(target: bigint) {
        super(...arguments)
        this.target = target
    }
    
    @method()
    public basicAssert(value: bigint) {
        // Simple assertion with message
        assert(value == this.target, 'Value does not match target')
        
        // If we reach here, assertion passed
        // No need for explicit return
    }
    
    @method()
    public multipleAssertions(x: bigint, y: bigint) {
        // All assertions must pass for success
        assert(x > 0n, 'X must be positive')
        assert(y > 0n, 'Y must be positive')
        assert(x + y == this.target, 'Sum incorrect')
        
        // Think of assertions as requirements that ALL must be met
    }
    
    @method()
    public conditionalLogic(value: bigint, useDouble: boolean) {
        // Conditional execution with assertions
        if (useDouble) {
            assert(value * 2n == this.target, 'Double does not match')
        } else {
            assert(value == this.target, 'Value does not match')
        }
    }
    
    @method()
    checkRange(value: bigint): boolean {
        // Helper methods can return booleans
        return value >= 0n && value <= 100n
    }
    
    @method()
    public complexValidation(value: bigint) {
        // Combine helper methods with assertions
        assert(this.checkRange(value), 'Value out of range')
        assert(value % 2n == 0n, 'Value must be even')
        assert(value == this.target, 'Value does not match target')
    }
}
```

### The Ending Rule

Public methods must end with an assertion or loop:

```typescript
// ✅ Good - ends with assertion
@method()
public unlock(x: bigint) {
    assert(x == this.target, 'Wrong value')
}

// ✅ Good - ends with assertion in all paths
@method()
public unlock2(x: bigint, useDouble: boolean) {
    if (useDouble) {
        assert(x * 2n == this.target, 'Wrong double')
    } else {
        assert(x == this.target, 'Wrong value')
    }
}

// ❌ Bad - might not end with assertion
@method()
public unlock3(x: bigint) {
    if (x > 10n) {
        assert(x == this.target, 'Wrong')
    }
    // Error: might reach here without assertion!
}
```

## 🏗️ Complete Example: Hash Puzzle

Let's put it all together with a complete example:

```typescript
import { SmartContract, method, prop, assert, ByteString, sha256 } from 'scrypt-ts'

/**
 * HashPuzzle - A contract that can be unlocked by providing
 * a value whose SHA256 hash matches the stored hash
 */
export class HashPuzzle extends SmartContract {
    // Store the expected hash on-chain
    @prop()
    readonly targetHash: ByteString
    
    /**
     * Constructor - Called off-chain when creating contract instance
     * @param targetHash - The SHA256 hash that must be matched
     */
    constructor(targetHash: ByteString) {
        super(...arguments)
        this.targetHash = targetHash
    }
    
    /**
     * Public unlock method - Called on-chain to spend the UTXO
     * @param preimage - The value that should hash to targetHash
     */
    @method()
    public unlock(preimage: ByteString) {
        // Calculate SHA256 hash of the provided preimage
        const calculatedHash = sha256(preimage)
        
        // Verify it matches our target hash
        // If this assertion fails, the transaction is rejected
        assert(
            calculatedHash == this.targetHash,
            'Hash mismatch: incorrect preimage provided'
        )
        
        // If we reach here, the preimage is correct!
        // The UTXO can be spent
    }
    
    /**
     * Alternative unlock with size restriction
     * @param preimage - The value to hash
     */
    @method()
    public unlockWithSizeCheck(preimage: ByteString) {
        // First check the size is reasonable
        assert(
            len(preimage) <= 100n,
            'Preimage too large'
        )
        
        // Then verify the hash
        assert(
            sha256(preimage) == this.targetHash,
            'Hash mismatch'
        )
    }
}

// Example usage (off-chain):
// const secret = toByteString('my-secret-value')
// const hash = sha256(secret)
// const puzzle = new HashPuzzle(hash)
// ... deploy puzzle ...
// ... later: unlock with secret ...
```

## 🎓 Key Takeaways

1. **Contracts are Classes** - Extend `SmartContract` base class
2. **Properties are Data** - Use `@prop` for on-chain storage
3. **Methods are Logic** - Use `@method` for on-chain functions
4. **Assertions are Control** - Use `assert()` to enforce rules
5. **Types are Limited** - Use only supported on-chain types
6. **Compilation is Magic** - TypeScript → Bitcoin Script

## 🚀 Next Steps

Now that you understand the basics:

1. Try modifying the examples above
2. Write your own simple contracts
3. Test them using the testing framework
4. Move on to [Built-in Functions](./built-ins.md) to expand your toolkit

Remember: Start simple and gradually increase complexity as you get comfortable with the concepts!

---

> 💡 **Practice Exercise**: Create a contract that locks funds until someone provides two numbers that multiply to a target value. Use what you learned about properties, methods, and assertions!