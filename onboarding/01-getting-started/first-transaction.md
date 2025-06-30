# Building BSV Applications with Metanet Desktop Wallet

## Why Metanet Desktop Wallet?

If you're looking for simple payments, wallets like HandCash or CentBee are perfect. But if you want to **build applications on BSV**, you need something more powerful.

The Metanet Desktop Wallet solves a critical problem: **applications shouldn't need to build their own wallet infrastructure**. Instead, they can connect to the user's Metanet Desktop Wallet and request actions through the WalletClient SDK.

## 🏗️ Understanding the Architecture

### Traditional Approach (Don't Do This)
```
Your App → Manages Keys → Signs Transactions → Broadcasts
         ↑ Security Risk!
```

### Metanet Desktop Wallet Approach (Do This)
```
Your App → WalletClient SDK → Metanet Desktop Wallet → Signs Safely → Broadcasts
         ↑ Keys Stay Secure!
```

## 🚀 Try It Yourself

To run these examples interactively:
1. Visit [fast.brc.dev](https://fast.brc.dev)
2. Copy any code snippet below
3. Click "Run Code" 
4. Your Metanet Desktop client will prompt for approval

### Your First Token Creation

This example creates a simple token in your wallet:

```typescript
import { WalletClient, Script } from '@bsv/sdk'

export async function createToken(runner) {

    // Connect to user's wallet
    const wallet = new WalletClient()

    // Create a token which represents an event ticket
    const response = await wallet.createAction({
      description: 'create an event ticket',
      outputs: [{
        satoshis: 1,
        lockingScript: Script.fromASM('OP_NOP').toHex(),
        basket: 'event tickets',
        outputDescription: 'event ticket'
      }]
    })

    return runner.log(response)
    
}
```

**What happens when you run this:**
1. Your browser connects to Metanet Desktop
2. Metanet Desktop shows you what the app wants to do
3. You approve or reject the action
4. If approved, the wallet creates and broadcasts the transaction

## 📦 Key Concepts

### Actions, Not Transactions

With Metanet Desktop Wallet, you don't build raw transactions. You describe **actions** you want to perform:

```typescript
// Instead of building a complex transaction...
// You just describe what you want:
const response = await wallet.createAction({
    description: 'What the user will see',
    outputs: [{
        satoshis: 1000,
        lockingScript: someScript,
        basket: 'my-app-tokens',
        outputDescription: 'My app token'
    }]
})
```

### Baskets: Organizing User Data

Baskets are like folders that organize different types of outputs:

```typescript
import { WalletClient } from '@bsv/sdk'

export async function listTokens(runner) {

    const wallet = new WalletClient()

    // List tokens in a specific basket
    const response = await wallet.listOutputs({
      basket: 'event tickets'
    })

    return runner.log(response)
    
}
```

## 💡 Real Application Example

Here's how a real BSV application posts a message to a public board:

```typescript
import { WalletClient, PushDrop, Utils, SecurityLevels, WalletProtocol, TopicBroadcaster, Transaction } from '@bsv/sdk'

export async function addTokenToOverlay(runner) {

    // Connect to user's wallet 
    const wallet = new WalletClient()

    const token = new PushDrop(wallet)

    // Your message
    const fields = []
    fields.push(Utils.toArray('Hello Overlay', 'utf8'))

    const protocolID: WalletProtocol = [SecurityLevels.Silent, 'hello world']
    const keyID = Date.now().toString()
    const counterparty = 'self'

    const script = await token.lock(fields, protocolID, keyID, counterparty)

    // Create a token containing the message
    const response = await wallet.createAction({
      description: 'Create Hello World Token',
      outputs: [{
        satoshis: 1,
        lockingScript: script.toHex(),
        basket: 'hello world',
        outputDescription: 'hello world token',
        customInstructions: keyID
      }]
    })

    // Capture the resulting transaction
    const tx = Transaction.fromBEEF(response.tx)

    runner.log(response)
    
    // Send to the hello world overlay service
    const overlay = new TopicBroadcaster(['tm_helloworld'])
    const overlayResponse = await tx.broadcast(overlay)

    runner.log(overlayResponse)
    
}
```

## 🌐 Working with Overlays

Overlays are specialized services that index and serve specific types of BSV data. Think of them as application-specific databases built on BSV:

```typescript
import { LookupResolver } from '@bsv/sdk'

export async function listHelloWorldTokens(runner) {

    const overlay = new LookupResolver()

    const response = await overlay.query({ 
        service: 'ls_helloworld', 
        query: {
            limit: 3,
            skip: 0,
            sortOrder: 'desc',
            message: 'Hello Overlay'
        } 
    }, 10000);

    runner.log(response)
    
}
```

## 📄 Distributed File Storage

BSV applications can store files using the UHRP (Universal Hash Resolution Protocol):

```typescript
import { WalletClient, StorageUploader, Utils } from '@bsv/sdk'

export async function upload(runner) {

    // Connect to user's wallet
    const wallet = new WalletClient()

    // Setup a client for uploading documents
    const uploader = new StorageUploader({
        wallet,
        storageURL: 'https://nanostore.babbage.systems'
    })

    const data = Utils.toArray('This can be any file buffer', 'utf8')

    const response = await uploader.publishFile({ 
        file: { 
            data, 
            type: 'text/plain' 
        }, 
        retentionPeriod: 180 // minutes
    })

    runner.log(response)
    
}
```

## 🔐 Security Benefits

### For Users
- **Keys Never Leave Wallet**: Applications can't steal your funds
- **Clear Approval**: See exactly what each action will do
- **Revokable Permissions**: Disconnect apps anytime
- **Transaction History**: Track what each app has done

### For Developers
- **No Key Management**: Let wallets handle the hard parts
- **User Trust**: Users feel safe using your app
- **Simplified Code**: Focus on your app logic, not wallet infrastructure
- **Broad Compatibility**: Works with any WalletClient-compatible wallet

## 🛠️ Building Your Own Application

### Step 1: Install the SDK
```bash
npm install @bsv/sdk
```

### Step 2: Connect to Wallet
```typescript
import { WalletClient } from '@bsv/sdk'

const wallet = new WalletClient()
```

### Step 3: Create Actions
```typescript
// Your app logic here
const response = await wallet.createAction({
    description: 'User-friendly description',
    outputs: [/* your outputs */]
})
```

### Step 4: Handle Responses
```typescript
if (response.status === 'success') {
    console.log('Transaction ID:', response.txid)
    // Update your app state
} else {
    // Handle rejection or error
}
```

## 🎯 Common Use Cases

### Social Media Apps
- Post content to BSV
- Like and comment transactions
- Tip other users
- Store media files

### Gaming Applications
- In-game item ownership
- High score records
- Tournament entries
- Achievement unlocks

### Business Tools
- Invoice generation
- Document timestamping
- Supply chain tracking
- Audit trails

### DeFi Applications
- Token creation and trading
- Liquidity pools
- Atomic swaps
- Escrow services

## 📚 Next Steps

1. **Try the Examples**: Go to [fast.brc.dev](https://fast.brc.dev) and run the code
2. **Read the SDK Docs**: [github.com/bitcoin-sv/ts-sdk](https://github.com/bitcoin-sv/ts-sdk)
3. **Join the Community**: [Discord](https://discord.gg/bsv) for help and discussions
4. **Build Something**: Start with a simple app and expand

## 🔗 Essential Resources

### Interactive Tools
- [fast.brc.dev](https://fast.brc.dev) - Run code snippets with wallet integration
- [WalletClient Playground](https://wallet.bsvblockchain.org/playground) - Test wallet features

### Documentation
- [WalletClient API Reference](https://docs.bsvblockchain.org/walletclient)
- [Overlay Services Guide](https://docs.bsvblockchain.org/overlays)
- [Token Protocols](https://docs.bsvblockchain.org/tokens)

### Example Applications
- [Hello World Overlay](https://github.com/bitcoin-sv/hello-world)
- [Token Examples](https://github.com/bitcoin-sv/token-examples)
- [Storage Examples](https://github.com/bitcoin-sv/storage-examples)

---

**Remember**: Metanet Desktop Wallet is for building applications, not for simple payments. It provides the infrastructure that lets you create rich, interactive BSV applications without the complexity of managing keys and wallets. Your users keep control of their funds while your app delivers amazing experiences!