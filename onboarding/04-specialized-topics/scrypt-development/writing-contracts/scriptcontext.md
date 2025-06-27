# Understanding ScriptContext

## 🎯 Accessing Transaction Data

ScriptContext provides your smart contract with access to the spending transaction's data. This is crucial for implementing complex spending conditions that depend on how the UTXO is being spent.

## 📚 The UTXO Model Context

Before diving into ScriptContext, let's understand where your contract executes:

```
Previous Transaction (Creating the UTXO):
┌─────────────────────────────┐
│ Outputs:                    │
│   [0] Your Contract (Lock)  │ ← This contains your compiled sCrypt
│       Amount: 10,000 sats  │
└─────────────────────────────┘

Current Transaction (Spending the UTXO):
┌─────────────────────────────┐
│ Inputs:                     │
│   [0] References above      │
│       Unlock Script         │ ← Your contract executes here!
│                            │
│ Outputs:                    │
│   [0] New destination      │
│   [1] Change output        │
└─────────────────────────────┘
```

## 🔍 What is ScriptContext?

ScriptContext (`this.ctx`) gives your contract visibility into:

1. **The spending transaction** - inputs, outputs, locktime
2. **The current UTXO** - which output is being spent
3. **Signature hash preimage** - raw transaction data for verification

```typescript
export class ContextExample extends SmartContract {
    @method()
    public unlock() {
        // Access the context - available in all public methods
        const ctx = this.ctx
        
        // Key information available:
        // - ctx.utxo: The UTXO being spent
        // - ctx.hashOutputs: Hash of all outputs
        // - ctx.hashPrevouts: Hash of all input references
        // - ctx.sigHashType: How the signature was calculated
        
        // Your contract logic can make decisions based on this data
    }
}
```

## 📊 Accessing Transaction Details

### The UTXO Being Spent

```typescript
export class UTXOInspector extends SmartContract {
    @method()
    public inspectUTXO() {
        // Information about the UTXO we're spending
        const utxo = this.ctx.utxo
        
        // The amount locked in this UTXO (in satoshis)
        const amount = utxo.value
        assert(amount >= 1000n, 'UTXO must have at least 1000 sats')
        
        // The locking script (your compiled contract)
        const lockingScript = utxo.script
        
        // This is useful for ensuring continuity in stateful contracts
    }
}
```

### Transaction Outputs

```typescript
export class OutputController extends SmartContract {
    @prop()
    readonly recipient: ByteString  // Address as bytes
    
    @method()
    public enforceOutput() {
        // Get hash of all outputs in spending transaction
        const outputsHash = this.ctx.hashOutputs
        
        // Build expected output
        const expectedAmount = 5000n
        const expectedScript = Utils.buildAddressScript(this.recipient)
        const expectedOutput = Utils.buildOutput(expectedScript, expectedAmount)
        
        // Ensure the spending transaction creates this exact output
        assert(
            hash256(expectedOutput) == outputsHash,
            'Transaction must send exactly 5000 sats to recipient'
        )
    }
}
```

### Advanced Output Control

```typescript
export class MultiOutputController extends SmartContract {
    @prop()
    readonly recipient1: PubKeyHash
    @prop()
    readonly recipient2: PubKeyHash
    
    @method()
    public splitFunds() {
        // Calculate split amounts
        const total = this.ctx.utxo.value
        const fee = 200n  // Transaction fee
        const half = (total - fee) / 2n
        
        // Build two outputs
        const output1 = Utils.buildPublicKeyHashOutput(this.recipient1, half)
        const output2 = Utils.buildPublicKeyHashOutput(this.recipient2, half)
        
        // Verify outputs match expectation
        const expectedOutputs = output1 + output2
        assert(
            hash256(expectedOutputs) == this.ctx.hashOutputs,
            'Must split funds equally between recipients'
        )
    }
}
```

## 🔐 SigHash Types

SigHash flags determine what parts of a transaction are signed:

```typescript
export class SigHashExamples extends SmartContract {
    @method()
    public demonstrateSigHash(sig: Sig, pubkey: PubKey) {
        // The signature includes a sigHashType byte at the end
        // This tells us what parts of the transaction were signed
        
        // Common SigHash types:
        // - SigHash.ALL (0x01): Sign all inputs and outputs (default)
        // - SigHash.NONE (0x02): Sign all inputs, no outputs
        // - SigHash.SINGLE (0x03): Sign all inputs, one output
        // - SigHash.ANYONECANPAY (0x80): Can be combined with above
        
        // Check what sigHashType was used
        const sigHashType = this.ctx.sigHashType
        
        // Ensure standard signature (ALL)
        assert(
            sigHashType == SigHash.ALL,
            'Must use SIGHASH_ALL'
        )
        
        // Verify signature
        assert(this.checkSig(sig, pubkey), 'Invalid signature')
    }
}
```

### Practical SigHash Usage

```typescript
export class FlexibleSpending extends SmartContract {
    @prop()
    readonly owner: PubKey
    
    @method()
    public flexibleUnlock(sig: Sig) {
        // Allow owner to add inputs (for fees) without invalidating signature
        const sigHashType = this.ctx.sigHashType
        
        // ANYONECANPAY allows adding more inputs
        const allowsAdditionalInputs = (sigHashType & SigHash.ANYONECANPAY) != 0
        
        assert(
            allowsAdditionalInputs,
            'Must allow additional inputs for fees'
        )
        
        // Verify owner signature
        assert(this.checkSig(sig, this.owner), 'Invalid owner signature')
    }
}
```

## 🔄 Stateful Contract Patterns

ScriptContext is essential for stateful contracts:

```typescript
export class Counter extends SmartContract {
    @prop(true)  // Mutable property
    count: bigint
    
    @method()
    public increment() {
        // Increment the counter
        this.count++
        
        // Ensure the new state is propagated
        const amount = this.ctx.utxo.value
        
        // Build output with updated state
        const output = this.buildStateOutput(amount)
        
        // Ensure this exact output is created
        assert(
            this.ctx.hashOutputs == hash256(output),
            'State must be preserved in output'
        )
    }
    
    @method()
    public incrementWithChange() {
        // Increment counter
        this.count++
        
        // Handle state output and change
        const amount = this.ctx.utxo.value - 1000n  // Deduct fee
        const stateOutput = this.buildStateOutput(amount)
        const changeOutput = this.buildChangeOutput()
        
        // Verify both outputs
        assert(
            this.ctx.hashOutputs == hash256(stateOutput + changeOutput),
            'Invalid outputs'
        )
    }
}
```

## 🛡️ Security Patterns

### Preventing UTXO Theft

```typescript
export class SecureVault extends SmartContract {
    @prop()
    readonly coldStorage: PubKeyHash
    
    @prop()
    readonly dailyLimit: bigint
    
    @method()
    public withdraw(amount: bigint, sig: Sig, pubkey: PubKey) {
        // Enforce daily limit
        assert(amount <= this.dailyLimit, 'Exceeds daily limit')
        
        // Ensure remaining funds go back to vault
        const remaining = this.ctx.utxo.value - amount - 200n  // fees
        
        if (remaining > 0n) {
            // Build outputs: withdrawal + vault
            const withdrawOutput = Utils.buildPublicKeyHashOutput(
                pubkey2addr(pubkey),
                amount
            )
            const vaultOutput = this.buildStateOutput(remaining)
            
            assert(
                this.ctx.hashOutputs == hash256(withdrawOutput + vaultOutput),
                'Invalid output structure'
            )
        }
        
        // Verify signature
        assert(this.checkSig(sig, pubkey), 'Invalid signature')
    }
}
```

### Transaction Introspection

```typescript
export class TransactionInspector extends SmartContract {
    @method()
    public enforceTransactionStructure() {
        // Access various transaction components
        const ctx = this.ctx
        
        // Version (usually 1 or 2)
        assert(ctx.version == 2n, 'Must use version 2 transactions')
        
        // Locktime (0 for no locktime)
        assert(ctx.locktime == 0n, 'No locktime allowed')
        
        // Input count (from hashPrevouts)
        // If hashPrevouts is not zero, there are inputs
        assert(ctx.hashPrevouts != toByteString(''), 'Must have inputs')
        
        // Enforce specific input/output patterns
        this.enforceCustomRules()
    }
    
    @method()
    enforceCustomRules() {
        // Your custom validation logic here
        assert(true, 'Custom rules')
    }
}
```

## 📝 Practical Examples

### Example 1: Payment Channel

```typescript
export class PaymentChannel extends SmartContract {
    @prop()
    readonly alice: PubKey
    
    @prop()
    readonly bob: PubKey
    
    @prop(true)
    balanceAlice: bigint
    
    @prop(true)
    balanceBob: bigint
    
    @method()
    public update(
        newBalanceAlice: bigint,
        newBalanceBob: bigint,
        sigAlice: Sig,
        sigBob: Sig
    ) {
        // Verify both parties agree
        assert(this.checkSig(sigAlice, this.alice), 'Invalid Alice signature')
        assert(this.checkSig(sigBob, this.bob), 'Invalid Bob signature')
        
        // Ensure total balance preserved
        const oldTotal = this.balanceAlice + this.balanceBob
        const newTotal = newBalanceAlice + newBalanceBob
        assert(oldTotal == newTotal, 'Total balance must be preserved')
        
        // Update balances
        this.balanceAlice = newBalanceAlice
        this.balanceBob = newBalanceBob
        
        // Propagate state
        const output = this.buildStateOutput(this.ctx.utxo.value)
        assert(this.ctx.hashOutputs == hash256(output), 'State not preserved')
    }
    
    @method()
    public close(sigAlice: Sig, sigBob: Sig) {
        // Both parties agree to close
        assert(this.checkSig(sigAlice, this.alice), 'Invalid Alice signature')
        assert(this.checkSig(sigBob, this.bob), 'Invalid Bob signature')
        
        // Build final outputs
        const outputAlice = Utils.buildPublicKeyHashOutput(
            pubKey2Addr(this.alice),
            this.balanceAlice
        )
        const outputBob = Utils.buildPublicKeyHashOutput(
            pubKey2Addr(this.bob),
            this.balanceBob
        )
        
        // Verify outputs
        assert(
            this.ctx.hashOutputs == hash256(outputAlice + outputBob),
            'Invalid closing outputs'
        )
    }
}
```

### Example 2: Crowdfunding

```typescript
export class Crowdfunding extends SmartContract {
    @prop()
    readonly target: bigint
    
    @prop()
    readonly deadline: bigint
    
    @prop()
    readonly beneficiary: PubKeyHash
    
    @method()
    public contribute() {
        // Anyone can contribute before deadline
        assert(
            this.ctx.locktime < this.deadline,
            'Contribution period ended'
        )
        
        // Accumulate contributions in single UTXO
        const currentAmount = this.ctx.utxo.value
        const outputs = this.buildStateOutput(currentAmount)
        
        assert(
            this.ctx.hashOutputs == hash256(outputs),
            'Invalid contribution'
        )
    }
    
    @method()
    public claim(beneficiarySig: Sig) {
        // Can only claim after deadline if target met
        assert(
            this.ctx.locktime >= this.deadline,
            'Cannot claim before deadline'
        )
        
        const raised = this.ctx.utxo.value
        assert(raised >= this.target, 'Target not met')
        
        // Send all funds to beneficiary
        const output = Utils.buildPublicKeyHashOutput(
            this.beneficiary,
            raised
        )
        
        assert(
            this.ctx.hashOutputs == hash256(output),
            'Must send all funds to beneficiary'
        )
    }
}
```

## 🎯 Best Practices

### 1. **Always Validate Outputs**
```typescript
// Don't just check signatures - verify outputs too!
const expectedOutput = this.buildExpectedOutput()
assert(this.ctx.hashOutputs == hash256(expectedOutput), 'Invalid outputs')
```

### 2. **Consider Fee Management**
```typescript
// Account for transaction fees
const inputAmount = this.ctx.utxo.value
const outputAmount = inputAmount - estimatedFee
assert(outputAmount > 0n, 'Insufficient funds for fee')
```

### 3. **Use Appropriate SigHash Types**
```typescript
// Choose SigHash based on your needs
// - ALL: Standard, signs everything
// - NONE: Flexible outputs (crowdfunding)
// - SINGLE: Pairs with specific output
// - ANYONECANPAY: Allows adding inputs
```

### 4. **State Preservation Pattern**
```typescript
// For stateful contracts, always:
// 1. Update state
this.state = newState

// 2. Build output with new state
const output = this.buildStateOutput(amount)

// 3. Verify output creation
assert(this.ctx.hashOutputs == hash256(output), 'State not preserved')
```

## 🚀 Next Steps

Now that you understand ScriptContext:

1. Practice building contracts that inspect transactions
2. Learn about [Stateful Contracts](./stateful-contracts.md) in detail
3. Explore advanced patterns like covenants
4. Study real-world contracts using ScriptContext

---

> 💡 **Challenge**: Create a "time-decay vault" contract where the daily withdrawal limit increases over time. Use ScriptContext to enforce that withdrawals follow the decay schedule!