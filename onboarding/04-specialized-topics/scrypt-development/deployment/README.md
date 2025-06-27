# Deploying Smart Contracts

## 🚀 From Code to Blockchain

You've written your smart contract - now it's time to deploy it to the BSV blockchain! This section covers everything from local testing to mainnet deployment, including the powerful LARS framework for streamlined deployments.

## 📚 Deployment Overview

The deployment process:

```
1. Write Contract (TypeScript)
        ↓
2. Compile to Bitcoin Script
        ↓
3. Create Deployment Transaction
        ↓
4. Broadcast to BSV Network
        ↓
5. Contract is Live!
```

## 🛠️ Basic Deployment

### Step 1: Compile Your Contract

```typescript
import { MyContract } from './contracts/myContract'

// Compile the contract to Bitcoin Script
await MyContract.compile()

// The compiled script is now available
console.log('Contract compiled successfully!')
```

### Step 2: Create Contract Instance

```typescript
// Create instance with constructor parameters
const instance = new MyContract(
    42n,  // Constructor parameter
    toByteString('Hello BSV!')
)

// Connect to a signer (wallet)
await instance.connect(signer)
```

### Step 3: Deploy to Network

```typescript
// Deploy the contract
const deployTx = await instance.deploy(1000)  // 1000 satoshis

console.log('Contract deployed!')
console.log('Transaction ID:', deployTx.id)
console.log('Contract Address:', instance.utxo)
```

## 🔑 Working with Signers

### What is a Signer?

A signer manages private keys and signs transactions:

```typescript
import { TestWallet, DefaultProvider } from 'scrypt-ts'

// For testing - uses a test wallet
const signer = new TestWallet(
    privateKey,
    new DefaultProvider({
        network: bsv.Networks.testnet
    })
)

// Connect signer to contract
await contract.connect(signer)
```

### Production Signers

```typescript
// For production, use secure key management
import { SensiletSigner } from '@sensible-contract/sdk'

// Sensilet wallet integration
const signer = new SensiletSigner()

// Request wallet connection
const { isAuthenticated, error } = await signer.requestAuth()
if (!isAuthenticated) {
    throw new Error('User rejected connection')
}

// Now you can deploy contracts
await contract.connect(signer)
```

## 📝 Complete Deployment Example

Here's a full deployment script:

```typescript
import { HashPuzzle } from './contracts/hashPuzzle'
import { sha256, toByteString } from 'scrypt-ts'
import { getDefaultSigner } from './utils/helper'

async function deployHashPuzzle() {
    try {
        // 1. Compile the contract
        await HashPuzzle.compile()
        console.log('Contract compiled ✓')
        
        // 2. Prepare deployment parameters
        const secret = toByteString('my-secret-value')
        const secretHash = sha256(secret)
        
        // 3. Create contract instance
        const puzzle = new HashPuzzle(secretHash)
        
        // 4. Connect signer
        const signer = await getDefaultSigner()
        await puzzle.connect(signer)
        console.log('Signer connected ✓')
        
        // 5. Deploy with initial funding
        const amount = 10000  // satoshis
        const deployTx = await puzzle.deploy(amount)
        
        console.log('Contract deployed! 🎉')
        console.log(`Transaction ID: ${deployTx.id}`)
        console.log(`View on WhatsOnChain:`)
        console.log(`https://whatsonchain.com/tx/${deployTx.id}`)
        
        // Save deployment info for later use
        const deploymentInfo = {
            txId: deployTx.id,
            outputIndex: 0,
            script: puzzle.lockingScript.toHex(),
            amount: amount
        }
        
        console.log('Deployment info:', deploymentInfo)
        
    } catch (error) {
        console.error('Deployment failed:', error)
    }
}

// Run deployment
deployHashPuzzle()
```

## 🏭 LARS - Local Development Environment

LARS (Local Automated Runtime System) provides a complete local development environment for BSV applications, automating Docker setup, service management, and contract compilation.

### What is LARS?

LARS is a powerful CLI tool that:
- **Automates** Docker-based environment setup with MySQL, MongoDB, and ngrok
- **Manages** BSV Overlay Services using `@bsv/overlay-express`
- **Handles** both backend and frontend development
- **Hot-reloads** contracts and code changes automatically
- **Bridges** seamlessly to CARS for cloud deployment

### Quick Start with LARS

```bash
# Install LARS in your project
npm install --save-dev @bsv/lars

# Run interactive setup
npx lars

# Or start directly
npx lars start
```

### LARS Configuration

LARS uses `deployment-info.json` for configuration:

```json
{
  "schema": "bsv-app",
  "schemaVersion": "1.0",
  "topicManagers": {
    "tm_voting": "./backend/src/topic-managers/VotingTopicManager.ts"
  },
  "lookupServices": {
    "ls_voting": {
      "serviceFactory": "./backend/src/lookup-services/VotingLookupServiceFactory.ts",
      "hydrateWith": "mongo"
    }
  },
  "contracts": {
    "language": "sCrypt",
    "baseDirectory": "./backend"
  },
  "configs": [{
    "name": "Local LARS",
    "network": "testnet",
    "provider": "LARS",
    "run": ["backend", "frontend"]
  }]
}
```

### Learn More About LARS

For complete LARS documentation including advanced features, troubleshooting, and best practices, see:

📖 **[Complete LARS Framework Guide](./lars-framework.md)**

## 🌐 Network Configuration

### Available Networks

```typescript
// Testnet (for development)
const testnet = bsv.Networks.testnet

// Mainnet (for production)
const mainnet = bsv.Networks.mainnet

// Local regtest (for testing)
const regtest = bsv.Networks.regtest
```

### Provider Configuration

```typescript
import { DefaultProvider, WhatsonchainProvider } from 'scrypt-ts'

// Default provider (uses multiple services)
const provider = new DefaultProvider({
    network: bsv.Networks.testnet
})

// WhatsOnChain provider
const wocProvider = new WhatsonchainProvider(
    bsv.Networks.mainnet
)

// Custom provider
const customProvider = new CustomProvider({
    url: 'https://api.mynode.com',
    headers: {
        'Authorization': `Bearer ${API_KEY}`
    }
})
```

## 📊 Monitoring Deployments

### Track Deployment Status

```typescript
async function monitorDeployment(txId: string) {
    const provider = new DefaultProvider()
    
    // Check if transaction is confirmed
    const tx = await provider.getTransaction(txId)
    console.log('Confirmations:', tx.confirmations)
    
    // Monitor for confirmations
    const confirmationTarget = 6
    
    while (tx.confirmations < confirmationTarget) {
        await new Promise(resolve => setTimeout(resolve, 60000))  // Wait 1 minute
        const updated = await provider.getTransaction(txId)
        console.log(`Confirmations: ${updated.confirmations}/${confirmationTarget}`)
    }
    
    console.log('Transaction fully confirmed!')
}
```

### Deployment Analytics

```typescript
class DeploymentTracker {
    async trackDeployment(contract: SmartContract, deployTx: Transaction) {
        const analytics = {
            contractType: contract.constructor.name,
            txId: deployTx.id,
            timestamp: new Date().toISOString(),
            network: contract.provider.network.name,
            gasUsed: deployTx.getFee(),
            contractSize: contract.lockingScript.toBuffer().length,
            fundingAmount: contract.balance
        }
        
        // Log to your analytics service
        await this.logAnalytics(analytics)
        
        // Store deployment history
        await this.saveDeploymentRecord(analytics)
        
        return analytics
    }
}
```

## 🔒 Security Best Practices

### 1. **Secure Key Management**

```typescript
// Never hardcode private keys!
// Bad:
const privateKey = 'L4rK1yDtCWekvXuE6oXD9jCYfFNV2cWRpVuPLBcCU2z8TrnZQVUG'

// Good:
const privateKey = process.env.DEPLOY_PRIVATE_KEY

// Better: Use hardware wallet or key management service
const signer = new HardwareWalletSigner()
```

### 2. **Validate Before Deployment**

```typescript
async function validateDeployment(contract: SmartContract) {
    // Check compilation
    if (!contract.isCompiled) {
        throw new Error('Contract not compiled')
    }
    
    // Verify network
    const network = await contract.provider.getNetwork()
    if (network.name !== expectedNetwork) {
        throw new Error(`Wrong network: ${network.name}`)
    }
    
    // Check balance
    const balance = await contract.signer.getBalance()
    if (balance < requiredAmount) {
        throw new Error('Insufficient funds')
    }
    
    return true
}
```

### 3. **Deployment Verification**

```typescript
async function verifyDeployment(txId: string, expectedScript: string) {
    const provider = new DefaultProvider()
    const tx = await provider.getTransaction(txId)
    
    // Verify the deployed script matches
    const output = tx.outputs[0]
    const deployedScript = output.script.toHex()
    
    if (deployedScript !== expectedScript) {
        throw new Error('Deployed script mismatch!')
    }
    
    console.log('Deployment verified ✓')
}
```

## 📋 Deployment Checklist

Before deploying to mainnet:

- [ ] Contract thoroughly tested on testnet
- [ ] Security audit completed
- [ ] Gas costs calculated and optimized
- [ ] Deployment parameters double-checked
- [ ] Backup of private keys secured
- [ ] Monitoring system ready
- [ ] Rollback plan prepared
- [ ] Documentation updated

## 🎯 Common Deployment Patterns

### Pattern 1: Factory Deployment

```typescript
class ContractFactory {
    template: typeof SmartContract
    
    async deployInstance(params: any, amount: bigint) {
        const instance = new this.template(...params)
        await instance.connect(signer)
        const tx = await instance.deploy(amount)
        
        return {
            instance,
            deployTx: tx,
            address: instance.address
        }
    }
}
```

### Pattern 2: Upgradeable Proxy

```typescript
class UpgradeableContract extends SmartContract {
    @prop()
    implementationHash: ByteString
    
    @prop()
    admin: PubKey
    
    @method()
    public upgrade(newImplHash: ByteString, adminSig: Sig) {
        assert(this.checkSig(adminSig, this.admin))
        this.implementationHash = newImplHash
        // ... state propagation
    }
}
```

## 🚀 Next Steps

Now that you can deploy contracts:

1. Learn about [Testing Contracts](../testing/README.md)
2. Master [Debugging Techniques](../debugging/README.md)
3. Explore [Frontend Integration](../frontend-integration/README.md)
4. Study [Production Best Practices](../best-practices.md)

---

> 💡 **Pro Tip**: Always deploy to testnet first! It's free and lets you catch issues before risking real funds on mainnet.