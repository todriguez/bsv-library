# Stateful Contracts

## 🔄 Contracts That Evolve Over Time

Traditional smart contracts on BSV are stateless - once deployed, they're immutable. Stateful contracts introduce a powerful pattern where contracts can maintain and update state across multiple transactions while preserving their logic.

## 📚 Stateless vs Stateful

### Stateless Contracts (Traditional)

```typescript
// Stateless: Same behavior forever
export class HashLock extends SmartContract {
    @prop()
    readonly hash: ByteString  // Fixed at deployment
    
    @method()
    public unlock(preimage: ByteString) {
        // Always requires the same preimage
        assert(sha256(preimage) == this.hash, 'Wrong preimage')
    }
}
```

### Stateful Contracts (Advanced)

```typescript
// Stateful: Can change over time
export class Counter extends SmartContract {
    @prop(true)  // true = mutable/stateful
    count: bigint  // Can increment with each transaction
    
    @method()
    public increment() {
        // State changes with each call
        this.count++
        
        // Must ensure state continues in next UTXO
        this.preserveState()
    }
}
```

## 🎯 How Stateful Contracts Work

The key insight: State is preserved by creating a new UTXO with updated values:

```
Transaction 1 (Deploy):
┌────────────────────────┐
│ Output:                │
│   Counter Contract     │
│   count = 0           │ ← Initial state
│   amount = 1000 sats  │
└────────────────────────┘

Transaction 2 (Increment):
┌────────────────────────┐
│ Input:                 │
│   Spends above         │
│                        │
│ Output:                │
│   Counter Contract     │
│   count = 1           │ ← Updated state!
│   amount = 1000 sats  │
└────────────────────────┘
```

## 🛠️ The @prop(true) Decorator

Mark properties as stateful with `@prop(true)`:

```typescript
export class StateExamples extends SmartContract {
    // Immutable properties (default)
    @prop()
    readonly owner: PubKey  // Cannot change
    
    // Mutable/stateful properties
    @prop(true)
    balance: bigint  // Can be updated
    
    @prop(true)
    lastUpdated: bigint  // Can track time
    
    @prop(true)
    status: ByteString  // Can change status
    
    constructor(owner: PubKey, initialBalance: bigint) {
        super(...arguments)
        this.owner = owner  // Set once
        this.balance = initialBalance  // Can change later
        this.lastUpdated = 0n
        this.status = toByteString('active')
    }
}
```

## 🔄 State Transitions

### Basic State Update Pattern

```typescript
export class SimpleCounter extends SmartContract {
    @prop(true)
    count: bigint
    
    @method()
    public increment() {
        // 1. Update the state
        this.count++
        
        // 2. Get current UTXO value
        const amount = this.ctx.utxo.value
        
        // 3. Build output with new state
        // This creates a new UTXO with count incremented
        const output = this.buildStateOutput(amount)
        
        // 4. Ensure this output is created
        assert(
            this.ctx.hashOutputs == hash256(output),
            'State update failed'
        )
    }
    
    @method()
    public add(value: bigint) {
        // Update by arbitrary amount
        this.count += value
        
        // Same pattern: update, build, verify
        const output = this.buildStateOutput(this.ctx.utxo.value)
        assert(this.ctx.hashOutputs == hash256(output))
    }
}
```

### Advanced State Transitions

```typescript
export class StateMachine extends SmartContract {
    // State machine with multiple states
    static readonly STATE_INIT = toByteString('init')
    static readonly STATE_ACTIVE = toByteString('active')
    static readonly STATE_PAUSED = toByteString('paused')
    static readonly STATE_CLOSED = toByteString('closed')
    
    @prop(true)
    currentState: ByteString
    
    @prop(true)
    stateData: bigint
    
    @prop()
    readonly admin: PubKey
    
    constructor(admin: PubKey) {
        super(...arguments)
        this.admin = admin
        this.currentState = StateMachine.STATE_INIT
        this.stateData = 0n
    }
    
    @method()
    public activate(adminSig: Sig) {
        // Verify admin signature
        assert(this.checkSig(adminSig, this.admin), 'Not admin')
        
        // State transition rules
        assert(
            this.currentState == StateMachine.STATE_INIT,
            'Can only activate from INIT state'
        )
        
        // Update state
        this.currentState = StateMachine.STATE_ACTIVE
        this.stateData = BigInt(Date.now())  // Timestamp
        
        // Preserve state in new UTXO
        this.propagateState()
    }
    
    @method()
    public pause(adminSig: Sig) {
        assert(this.checkSig(adminSig, this.admin), 'Not admin')
        assert(
            this.currentState == StateMachine.STATE_ACTIVE,
            'Can only pause from ACTIVE state'
        )
        
        this.currentState = StateMachine.STATE_PAUSED
        this.propagateState()
    }
    
    @method()
    propagateState() {
        // Helper to ensure state continues
        const output = this.buildStateOutput(this.ctx.utxo.value)
        assert(this.ctx.hashOutputs == hash256(output))
    }
}
```

## 💰 Handling Value in Stateful Contracts

### Preserving Value

```typescript
export class ValuePreserver extends SmartContract {
    @prop(true)
    data: bigint
    
    @method()
    public update(newData: bigint) {
        this.data = newData
        
        // Preserve the exact same amount
        const amount = this.ctx.utxo.value
        const output = this.buildStateOutput(amount)
        
        assert(this.ctx.hashOutputs == hash256(output))
    }
}
```

### Allowing Deposits

```typescript
export class DepositBox extends SmartContract {
    @prop(true)
    totalDeposited: bigint
    
    @prop()
    readonly owner: PubKey
    
    @method()
    public deposit() {
        // Anyone can add funds
        const previousAmount = this.ctx.utxo.value
        const currentAmount = this.ctx.utxo.value  // Will be more due to deposit
        
        // Track total deposited
        this.totalDeposited += (currentAmount - previousAmount)
        
        // Continue with new balance
        const output = this.buildStateOutput(currentAmount)
        assert(this.ctx.hashOutputs == hash256(output))
    }
    
    @method()
    public withdraw(amount: bigint, ownerSig: Sig) {
        // Only owner can withdraw
        assert(this.checkSig(ownerSig, this.owner), 'Not owner')
        
        const currentBalance = this.ctx.utxo.value
        assert(amount <= currentBalance, 'Insufficient funds')
        
        // Build withdrawal output
        const withdrawOutput = Utils.buildPublicKeyHashOutput(
            pubKey2Addr(this.owner),
            amount
        )
        
        // Continue contract with remaining balance
        const remainingBalance = currentBalance - amount
        const stateOutput = this.buildStateOutput(remainingBalance)
        
        // Verify both outputs created
        assert(
            this.ctx.hashOutputs == hash256(withdrawOutput + stateOutput)
        )
    }
}
```

## 🎮 Real-World Example: Tic-Tac-Toe

Here's a complete stateful contract implementing a game:

```typescript
export class TicTacToe extends SmartContract {
    @prop(true)
    board: FixedArray<bigint, 9>  // 0=empty, 1=X, 2=O
    
    @prop(true)
    isXTurn: boolean
    
    @prop(true)
    gameOver: boolean
    
    @prop()
    readonly playerX: PubKey
    
    @prop()
    readonly playerO: PubKey
    
    constructor(playerX: PubKey, playerO: PubKey) {
        super(...arguments)
        this.playerX = playerX
        this.playerO = playerO
        this.board = [0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n, 0n]
        this.isXTurn = true
        this.gameOver = false
    }
    
    @method()
    public makeMove(position: bigint, sig: Sig) {
        // Game must be ongoing
        assert(!this.gameOver, 'Game is over')
        
        // Validate position
        assert(position >= 0n && position < 9n, 'Invalid position')
        assert(this.board[Number(position)] == 0n, 'Position taken')
        
        // Verify correct player
        if (this.isXTurn) {
            assert(this.checkSig(sig, this.playerX), 'Not player X')
            this.board[Number(position)] = 1n  // X
        } else {
            assert(this.checkSig(sig, this.playerO), 'Not player O')
            this.board[Number(position)] = 2n  // O
        }
        
        // Check for winner
        if (this.checkWinner()) {
            this.gameOver = true
            
            // Winner takes all
            const winner = this.isXTurn ? this.playerX : this.playerO
            const winnerOutput = Utils.buildPublicKeyHashOutput(
                pubKey2Addr(winner),
                this.ctx.utxo.value
            )
            
            assert(this.ctx.hashOutputs == hash256(winnerOutput))
        } else {
            // Game continues
            this.isXTurn = !this.isXTurn
            
            // Check for draw
            if (this.isBoardFull()) {
                this.gameOver = true
                // Split the pot on draw
                this.splitPot()
            } else {
                // Continue game
                const output = this.buildStateOutput(this.ctx.utxo.value)
                assert(this.ctx.hashOutputs == hash256(output))
            }
        }
    }
    
    @method()
    checkWinner(): boolean {
        // Check all winning combinations
        const lines = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],  // Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8],  // Columns
            [0, 4, 8], [2, 4, 6]              // Diagonals
        ]
        
        for (let i = 0; i < 8; i++) {
            const [a, b, c] = lines[i]
            if (this.board[a] != 0n &&
                this.board[a] == this.board[b] &&
                this.board[a] == this.board[c]) {
                return true
            }
        }
        return false
    }
    
    @method()
    isBoardFull(): boolean {
        for (let i = 0; i < 9; i++) {
            if (this.board[i] == 0n) {
                return false
            }
        }
        return true
    }
    
    @method()
    splitPot() {
        const half = this.ctx.utxo.value / 2n
        
        const outputX = Utils.buildPublicKeyHashOutput(
            pubKey2Addr(this.playerX),
            half
        )
        const outputO = Utils.buildPublicKeyHashOutput(
            pubKey2Addr(this.playerO),
            half
        )
        
        assert(this.ctx.hashOutputs == hash256(outputX + outputO))
    }
}
```

## 🔧 State Management Patterns

### Pattern 1: Incremental Updates

```typescript
export class IncrementalState extends SmartContract {
    @prop(true)
    counter: bigint
    
    @prop(true)
    lastAction: ByteString
    
    @prop(true)
    history: FixedArray<bigint, 10>  // Last 10 values
    
    @method()
    public recordAction(action: ByteString, value: bigint) {
        // Shift history
        for (let i = 9; i > 0; i--) {
            this.history[i] = this.history[i-1]
        }
        this.history[0] = value
        
        // Update other state
        this.counter++
        this.lastAction = action
        
        // Propagate
        this.propagateState()
    }
}
```

### Pattern 2: Conditional State Transitions

```typescript
export class ConditionalStateMachine extends SmartContract {
    @prop(true)
    stage: bigint  // 0=init, 1=active, 2=complete
    
    @prop(true)
    participants: bigint
    
    @prop()
    readonly maxParticipants: bigint
    
    @method()
    public join() {
        // Can only join in init stage
        assert(this.stage == 0n, 'Not in joining stage')
        
        this.participants++
        
        // Auto-transition when full
        if (this.participants >= this.maxParticipants) {
            this.stage = 1n  // Move to active
        }
        
        this.propagateState()
    }
}
```

### Pattern 3: Time-Based State

```typescript
export class TimedAuction extends SmartContract {
    @prop(true)
    highestBid: bigint
    
    @prop(true)
    highestBidder: PubKey
    
    @prop()
    readonly endTime: bigint  // Block height
    
    @method()
    public bid(bidder: PubKey, bidSig: Sig) {
        // Auction must be active
        assert(this.ctx.locktime < this.endTime, 'Auction ended')
        
        // New bid must be higher
        const newBid = this.ctx.utxo.value
        assert(newBid > this.highestBid, 'Bid too low')
        
        // Refund previous bidder
        if (this.highestBid > 0n) {
            const refundOutput = Utils.buildPublicKeyHashOutput(
                pubKey2Addr(this.highestBidder),
                this.highestBid
            )
            
            // Update state
            this.highestBid = newBid
            this.highestBidder = bidder
            
            // Continue auction with new bid
            const stateOutput = this.buildStateOutput(newBid)
            
            assert(
                this.ctx.hashOutputs == hash256(refundOutput + stateOutput)
            )
        } else {
            // First bid
            this.highestBid = newBid
            this.highestBidder = bidder
            this.propagateState()
        }
    }
}
```

## 🎓 Best Practices

### 1. **Always Validate State Transitions**
```typescript
// Good: Explicit state validation
assert(this.currentState == STATE_A, 'Invalid state for transition')
this.currentState = STATE_B

// Bad: No validation
this.currentState = STATE_B  // Could be called from wrong state!
```

### 2. **Handle Edge Cases**
```typescript
// Consider empty states, overflows, etc.
if (this.counter < MAX_VALUE) {
    this.counter++
} else {
    assert(false, 'Counter overflow')
}
```

### 3. **Minimize State Size**
```typescript
// Good: Compact state representation
@prop(true)
flags: bigint  // Bit flags for multiple booleans

// Less efficient: Multiple separate properties
@prop(true)
flag1: boolean
@prop(true)
flag2: boolean
```

### 4. **Design for Gas Efficiency**
```typescript
// Minimize operations in state transitions
// Pre-calculate when possible
// Use efficient data structures
```

## 🚀 Advanced Patterns

### Token Implementation

```typescript
export class SimpleToken extends SmartContract {
    @prop(true)
    amount: bigint
    
    @prop()
    readonly owner: PubKey
    
    @method()
    public transfer(
        recipientAmount: bigint,
        recipientPubKey: PubKey,
        ownerSig: Sig
    ) {
        assert(this.checkSig(ownerSig, this.owner), 'Not owner')
        assert(recipientAmount <= this.amount, 'Insufficient balance')
        
        // Create recipient's token
        const recipientToken = new SimpleToken(recipientAmount, recipientPubKey)
        const recipientOutput = recipientToken.buildStateOutput(recipientAmount)
        
        // Update sender's balance
        this.amount -= recipientAmount
        
        if (this.amount > 0n) {
            // Continue with remaining balance
            const senderOutput = this.buildStateOutput(this.amount)
            assert(
                this.ctx.hashOutputs == hash256(recipientOutput + senderOutput)
            )
        } else {
            // All tokens transferred
            assert(this.ctx.hashOutputs == hash256(recipientOutput))
        }
    }
}
```

## 🎯 Summary

Stateful contracts enable:
- **Persistent State** - Data that survives across transactions
- **Complex Applications** - Games, tokens, auctions, channels
- **Dynamic Behavior** - Contracts that adapt over time
- **Multi-party Interactions** - Coordinated state updates

Key concepts:
1. Use `@prop(true)` for mutable state
2. Always propagate state to new UTXOs
3. Validate all state transitions
4. Consider gas costs and efficiency

## 🚦 Next Steps

You've mastered the core concepts! Now:

1. Build your own stateful contracts
2. Study the [examples repository](https://github.com/sCrypt-Inc/boilerplate)
3. Learn about [deployment strategies](../deployment/README.md)
4. Explore advanced patterns like state channels

---

> 💡 **Challenge**: Create a "multi-signature wallet" contract where owners can propose and vote on transactions. Use stateful properties to track proposals and votes!