# Built-in Functions

## 🛠️ Your Smart Contract Toolkit

sCrypt provides a comprehensive set of built-in functions for common smart contract operations. These functions compile directly to efficient Bitcoin Script opcodes, giving you powerful tools while maintaining on-chain efficiency.

## 📚 Function Categories

1. **Assertions & Control Flow** - Enforce contract rules
2. **Cryptographic Functions** - Hashing and signatures
3. **Mathematical Operations** - Arithmetic and comparisons
4. **Byte Operations** - Manipulate ByteStrings
5. **Utility Functions** - Type conversions and helpers

## ✅ Assertions and Control Flow

### assert()

The fundamental building block of smart contract logic:

```typescript
export class AssertionExamples extends SmartContract {
    @prop()
    readonly target: bigint = 42n
    
    @method()
    public demonstrateAssertions(value: bigint) {
        // Basic assertion - transaction fails if condition is false
        assert(value > 0n, 'Value must be positive')
        
        // Compound conditions
        assert(value > 0n && value < 100n, 'Value must be between 1 and 99')
        
        // Using with equality
        assert(value == this.target, 'Value must equal target')
        
        // Chain multiple assertions
        assert(value > 0n, 'Must be positive')
        assert(value % 2n == 0n, 'Must be even')
        assert(value == this.target, 'Must equal target')
    }
}
```

### Why Assertions Matter

```typescript
// Traditional programming returns values
function traditional(x: number): boolean {
    if (x > 10) {
        return true
    }
    return false
}

// Smart contracts use assertions
@method()
public smartContract(x: bigint) {
    // No return value - assertion determines success/failure
    assert(x > 10n, 'X must be greater than 10')
    // Transaction succeeds only if assertion passes
}
```

### exit()

Force an early exit (always fails):

```typescript
@method()
public conditionalExit(value: bigint, shouldExit: boolean) {
    if (shouldExit) {
        // Force failure with custom message
        exit(false, 'Early exit requested')
    }
    
    // Continue normal execution
    assert(value == this.target, 'Wrong value')
}
```

## 🔐 Cryptographic Functions

### Hash Functions

```typescript
export class HashingExamples extends SmartContract {
    @method()
    public demonstrateHashing(data: ByteString) {
        // SHA-256 - Most common, 32-byte output
        const sha256Hash = sha256(data)
        
        // Double SHA-256 - Used for block hashes
        const hash256Result = hash256(data)
        // Equivalent to: sha256(sha256(data))
        
        // SHA-1 - Legacy, 20-byte output (avoid for new designs)
        const sha1Hash = sha1(data)
        
        // RIPEMD-160 - 20-byte output
        const ripemd160Hash = ripemd160(data)
        
        // HASH160 - SHA256 then RIPEMD160 (used in addresses)
        const hash160Result = hash160(data)
        // Equivalent to: ripemd160(sha256(data))
        
        // Example: Verify a hash
        const expectedHash = toByteString('0x2c26b46b68ffc68ff99b453c1d30413413422d706483bfa0f98a5e886266e7ae')
        assert(sha256(data) == expectedHash, 'Hash mismatch')
    }
    
    @method()
    public hashChallenge(
        preimage1: ByteString,
        preimage2: ByteString,
        targetHash: ByteString
    ) {
        // Combine multiple preimages
        const combined = preimage1 + preimage2
        
        // Verify combined hash
        assert(sha256(combined) == targetHash, 'Invalid preimages')
    }
}
```

### Signature Verification

```typescript
export class SignatureExamples extends SmartContract {
    @prop()
    readonly owner: PubKey
    
    @prop()
    readonly backup: PubKey
    
    constructor(owner: PubKey, backup: PubKey) {
        super(...arguments)
        this.owner = owner
        this.backup = backup
    }
    
    @method()
    public singleSigUnlock(sig: Sig) {
        // Verify signature matches the owner's public key
        // checkSig uses ECDSA verification
        assert(this.checkSig(sig, this.owner), 'Invalid owner signature')
    }
    
    @method()
    public multiSigUnlock(ownerSig: Sig, backupSig: Sig) {
        // Require both signatures
        assert(this.checkSig(ownerSig, this.owner), 'Invalid owner signature')
        assert(this.checkSig(backupSig, this.backup), 'Invalid backup signature')
    }
    
    @method()
    public flexibleUnlock(sig: Sig, pubkey: PubKey) {
        // Check if signer is authorized
        const isOwner = (pubkey == this.owner)
        const isBackup = (pubkey == this.backup)
        
        assert(isOwner || isBackup, 'Unauthorized public key')
        assert(this.checkSig(sig, pubkey), 'Invalid signature')
    }
}
```

### Advanced Signature Checking

```typescript
export class AdvancedSigExamples extends SmartContract {
    @method()
    public checkMultiSigExample(sigs: FixedArray<Sig, 3>, pubkeys: FixedArray<PubKey, 3>) {
        // checkMultiSig verifies multiple signatures at once
        // All signatures must be valid for success
        assert(this.checkMultiSig(sigs, pubkeys), 'Invalid signatures')
    }
    
    @method()
    public thresholdSig(
        sigs: FixedArray<Sig, 2>,
        pubkeys: FixedArray<PubKey, 3>
    ) {
        // Implement 2-of-3 multisig manually
        let validSigs = 0n
        
        // Check each signature against each pubkey
        for (let i = 0; i < 2; i++) {
            for (let j = 0; j < 3; j++) {
                if (this.checkSig(sigs[i], pubkeys[j])) {
                    validSigs++
                    break  // Move to next signature
                }
            }
        }
        
        assert(validSigs >= 2n, 'Need at least 2 valid signatures')
    }
}
```

## 🔢 Mathematical Operations

### Arithmetic Functions

```typescript
export class MathExamples extends SmartContract {
    @method()
    public demonstrateMath(a: bigint, b: bigint) {
        // Basic arithmetic works as expected
        const sum = a + b
        const difference = a - b
        const product = a * b
        const quotient = a / b  // Integer division
        const remainder = a % b  // Modulo
        
        // Absolute value
        const absValue = abs(a - b)
        
        // Min and max
        const minimum = min(a, b)
        const maximum = max(a, b)
        
        // Comparison operators
        assert(a > 0n, 'A must be positive')
        assert(b != 0n, 'B cannot be zero')
        assert(a >= b, 'A must be >= B')
        
        // Within range check
        const value = 50n
        assert(within(value, 0n, 100n), 'Value must be 0-100')
        // Equivalent to: value >= 0n && value <= 100n
    }
    
    @method()
    public squareRoot(n: bigint) {
        // Calculate integer square root
        const root = sqrt(n)
        
        // Verify it's correct (within integer precision)
        assert(root * root <= n, 'Root too large')
        assert((root + 1n) * (root + 1n) > n, 'Root too small')
    }
}
```

### Advanced Math Examples

```typescript
export class AdvancedMathContract extends SmartContract {
    @method()
    public demonstrateAdvancedMath(x: bigint, y: bigint, z: bigint) {
        // Chained operations
        const result = abs(x - y) + min(y, z) * 2n
        
        // Percentage calculation (using integer math)
        const total = 1000n
        const percentage = 15n  // 15%
        const amount = (total * percentage) / 100n  // = 150
        
        // Safe division (avoid division by zero)
        const safeDivide = y != 0n ? x / y : 0n
        
        // Power calculation (manual since no built-in power)
        let power = 1n
        for (let i = 0n; i < 3n; i++) {
            power = power * x  // x^3
        }
        
        assert(power == x * x * x, 'Power calculation check')
    }
}
```

## 📝 ByteString Operations

### Basic Byte Operations

```typescript
export class ByteOperations extends SmartContract {
    @method()
    public demonstrateByteOps(data: ByteString) {
        // Get length in bytes
        const dataLength = len(data)
        assert(dataLength > 0n, 'Data cannot be empty')
        
        // Concatenation
        const prefix = toByteString('0x48656c6c6f')  // "Hello"
        const combined = prefix + data
        
        // Slicing - extract portion of bytes
        const firstByte = slice(data, 0n, 1n)  // Get first byte
        const lastByte = slice(data, dataLength - 1n, dataLength)
        
        // Reverse bytes
        const reversed = reverseBytes(data, 32n)  // Reverse 32 bytes
        
        // Check equality
        const expected = toByteString('0x1234')
        assert(data == expected, 'Data mismatch')
    }
    
    @method()
    public packingExample() {
        // Pack multiple values into ByteString
        const a = 42n
        const b = true
        const c = toByteString('hello')
        
        // Use Scrypt.ts pack function (compile-time known size)
        // This efficiently encodes multiple values
        const packed = pack(a, b, c)
        
        // Later can unpack (in another method)
        // const [a2, b2, c2] = unpack(packed)
    }
}
```

### Advanced Byte Manipulation

```typescript
export class AdvancedByteOps extends SmartContract {
    @method()
    public bytePatternMatching(data: ByteString, pattern: ByteString) {
        // Check if data starts with pattern
        const patternLen = len(pattern)
        const prefix = slice(data, 0n, patternLen)
        assert(prefix == pattern, 'Pattern not found at start')
        
        // Extract and validate parts
        const header = slice(data, 0n, 4n)  // 4-byte header
        const body = slice(data, 4n, len(data) - 4n)  // Skip header and checksum
        const checksum = slice(data, len(data) - 4n, len(data))  // Last 4 bytes
        
        // Verify checksum
        const calculatedChecksum = slice(hash256(header + body), 0n, 4n)
        assert(checksum == calculatedChecksum, 'Invalid checksum')
    }
}
```

## 🔄 Type Conversions

### Converting Between Types

```typescript
export class TypeConversions extends SmartContract {
    @method()
    public demonstrateConversions() {
        // String to ByteString
        const text = toByteString('Hello BSV!')  // UTF-8 encoding
        const hex = toByteString('0xdeadbeef')   // Hex encoding
        const empty = toByteString('')           // Empty bytes
        
        // Number to ByteString (little-endian)
        const numBytes = int2ByteString(42n)     // Number as bytes
        
        // ByteString to Number (for small values)
        const num = byteString2Int(numBytes)     // Back to number
        
        // Boolean conversions
        const boolBytes = bool2ByteString(true)  // 0x01 for true
        
        // Size conversions
        const paddedBytes = int2ByteString(42n, 4n)  // Pad to 4 bytes
    }
    
    @method()
    public practicalConversions(userInput: ByteString) {
        // Example: Parse protocol message
        // Format: [1 byte type][4 bytes length][data]
        
        const msgType = byteString2Int(slice(userInput, 0n, 1n))
        const dataLen = byteString2Int(slice(userInput, 1n, 5n))
        const data = slice(userInput, 5n, 5n + dataLen)
        
        // Validate message type
        assert(msgType >= 1n && msgType <= 10n, 'Invalid message type')
        assert(len(data) == dataLen, 'Length mismatch')
    }
}
```

## 🎯 Practical Examples

### Example 1: Time-Locked Contract

```typescript
export class TimeLock extends SmartContract {
    @prop()
    readonly recipient: PubKey
    
    @prop()
    readonly lockUntil: bigint  // Block height
    
    constructor(recipient: PubKey, lockUntil: bigint) {
        super(...arguments)
        this.recipient = recipient
        this.lockUntil = lockUntil
    }
    
    @method()
    public unlock(sig: Sig) {
        // Check time condition using ScriptContext
        // (Assuming we have access to current block height)
        const currentHeight = this.ctx.blockHeight
        assert(currentHeight >= this.lockUntil, 'Still time-locked')
        
        // Verify recipient signature
        assert(this.checkSig(sig, this.recipient), 'Invalid signature')
    }
}
```

### Example 2: Atomic Swap

```typescript
export class AtomicSwap extends SmartContract {
    @prop()
    readonly secretHash: ByteString
    
    @prop()
    readonly timeout: bigint
    
    @prop()
    readonly alice: PubKey
    
    @prop()
    readonly bob: PubKey
    
    @method()
    public claimWithSecret(secret: ByteString, bobSig: Sig) {
        // Bob claims with secret
        assert(sha256(secret) == this.secretHash, 'Invalid secret')
        assert(this.checkSig(bobSig, this.bob), 'Invalid Bob signature')
    }
    
    @method()
    public refund(aliceSig: Sig) {
        // Alice reclaims after timeout
        assert(this.ctx.blockHeight >= this.timeout, 'Too early for refund')
        assert(this.checkSig(aliceSig, this.alice), 'Invalid Alice signature')
    }
}
```

### Example 3: Voting Contract

```typescript
export class SimpleVoting extends SmartContract {
    @prop()
    readonly voters: FixedArray<PubKey, 5>
    
    @prop()
    readonly threshold: bigint
    
    @method()
    public vote(sigs: FixedArray<Sig, 5>, votes: FixedArray<boolean, 5>) {
        let yesVotes = 0n
        
        // Count valid yes votes
        for (let i = 0; i < 5; i++) {
            if (votes[i] && this.checkSig(sigs[i], this.voters[i])) {
                yesVotes++
            }
        }
        
        // Check if threshold met
        assert(yesVotes >= this.threshold, 'Insufficient yes votes')
    }
}
```

## 📖 Complete Function Reference

### Assertion Functions
- `assert(condition, message)` - Enforce a requirement
- `exit(status, message)` - Force exit with status

### Hash Functions
- `sha1(ByteString): ByteString` - SHA-1 hash
- `sha256(ByteString): ByteString` - SHA-256 hash
- `hash256(ByteString): ByteString` - Double SHA-256
- `ripemd160(ByteString): ByteString` - RIPEMD-160 hash
- `hash160(ByteString): ByteString` - SHA-256 + RIPEMD-160

### Signature Functions
- `checkSig(Sig, PubKey): boolean` - Verify single signature
- `checkMultiSig(Sig[], PubKey[]): boolean` - Verify multiple signatures

### Math Functions
- `abs(bigint): bigint` - Absolute value
- `min(bigint, bigint): bigint` - Minimum value
- `max(bigint, bigint): bigint` - Maximum value
- `within(bigint, bigint, bigint): boolean` - Range check
- `sqrt(bigint): bigint` - Integer square root

### ByteString Functions
- `len(ByteString): bigint` - Get byte length
- `slice(ByteString, bigint, bigint): ByteString` - Extract substring
- `reverseBytes(ByteString, bigint): ByteString` - Reverse byte order
- `pack(...): ByteString` - Pack multiple values

### Conversion Functions
- `int2ByteString(bigint, bigint?): ByteString` - Number to bytes
- `byteString2Int(ByteString): bigint` - Bytes to number
- `toByteString(string): ByteString` - String/hex to bytes
- `bool2ByteString(boolean): ByteString` - Boolean to bytes

## 🎓 Best Practices

1. **Use Type-Safe Functions** - Prefer built-ins over manual operations
2. **Check Bounds** - Always validate array indices and slice ranges
3. **Handle Edge Cases** - Empty bytes, zero values, etc.
4. **Optimize Gas** - Some operations are more expensive than others
5. **Test Thoroughly** - Built-ins can fail with invalid inputs

## 🚀 Next Steps

Now that you know the built-in functions:

1. Practice using different functions in contracts
2. Study [ScriptContext](./scriptcontext.md) for transaction data
3. Learn about [Stateful Contracts](./stateful-contracts.md)
4. Build more complex contracts combining multiple functions

---

> 💡 **Challenge**: Create a contract that implements a commit-reveal scheme using hash functions and time locks. Users first commit a hashed value, then reveal it after a certain block height!