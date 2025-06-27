# The BSV Submodule

## 🔧 Your Blockchain Toolkit

sCrypt provides a powerful `bsv` submodule that handles all the low-level blockchain operations you need. Think of it as your Swiss Army knife for BSV development - it manages keys, addresses, transactions, and cryptographic functions so you can focus on building great smart contracts.

## Importing the BSV Submodule

```typescript
import { bsv } from 'scrypt-ts'

// The bsv object contains all the utilities you need
// Let's explore what's inside!
```

## 🔑 Private Keys

Private keys are the foundation of BSV security. They're like the master password to your digital vault - keep them secret!

### Generating Private Keys

```typescript
// Method 1: Generate a random private key for mainnet
const privKey = bsv.PrivateKey.fromRandom()
console.log(privKey.toString())
// Example output: "L4rK1yDtCWekvXuE6oXD9jCYfFNV2cWRpVuPLBcCU2z8TrnZQVUG"
// This is WIF (Wallet Import Format) - a standard way to represent private keys

// Method 2: Generate for testnet (for development)
const testnetPrivKey = bsv.PrivateKey.fromRandom(bsv.Networks.testnet)
console.log(testnetPrivKey.toString())
// Example output: "cVBDG2FnHrZxVZGDZMztkNTJxYC1h3s9AQjUGSebvot5mJRHDMfR"
// Notice it starts with 'c' for testnet instead of 'L' for mainnet

// Method 3: Import existing private key from WIF string
// NEVER hardcode real private keys in your code!
const importedKey = bsv.PrivateKey.fromWIF('L4rK1yDtCWekvXuE6oXD9jCYfFNV2cWRpVuPLBcCU2z8TrnZQVUG')
```

### Understanding Private Keys

```typescript
// A private key is just a very large random number
// Let's see what's under the hood

const myKey = bsv.PrivateKey.fromRandom()

// The actual number (32 bytes = 256 bits)
console.log('Private key as hex:', myKey.toHex())
// Example: "e9873d79c6d87dc0fb6a5778633389f4453213303da61f20bd67fc233aa33262"

// The network it belongs to
console.log('Network:', myKey.network.name)  // "mainnet" or "testnet"

// Convert between formats
const wif = myKey.toWIF()        // Wallet Import Format (standard)
const hex = myKey.toHex()        // Raw hexadecimal
const bn = myKey.toBigNumber()   // As a BigNumber for math operations
```

## 🔓 Public Keys

Public keys are derived from private keys using elliptic curve mathematics. They're safe to share and are used to create BSV addresses.

### Deriving Public Keys

```typescript
// Always derived from a private key
const privKey = bsv.PrivateKey.fromRandom()
const pubKey = privKey.toPublicKey()

// What can we do with a public key?
console.log('Public key (hex):', pubKey.toHex())
// Example: "02b4632d08485ff1df2db55b9dafd23347d1c47a457072a1e87be26896549a8737"

// Public keys in BSV use the secp256k1 elliptic curve
// They represent a point (x, y) on this curve
console.log('X coordinate:', pubKey.point.x.toString(16))
console.log('Y coordinate:', pubKey.point.y.toString(16))

// Compressed vs uncompressed formats
const compressed = pubKey.toHex()              // 33 bytes (default)
const uncompressed = pubKey.toHex(false)       // 65 bytes
console.log('Compressed (33 bytes):', compressed)
console.log('Uncompressed (65 bytes):', uncompressed)
```

### Public Key Operations

```typescript
// You can create a public key directly if you have the hex
const pubKeyHex = '02b4632d08485ff1df2db55b9dafd23347d1c47a457072a1e87be26896549a8737'
const pubKey2 = bsv.PublicKey.fromHex(pubKeyHex)

// Verify it's valid
console.log('Is valid?', bsv.PublicKey.isValid(pubKeyHex))  // true

// Get the address for receiving payments
const address = pubKey2.toAddress()
console.log('Address:', address.toString())
// Example: "1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm"
```

## 📬 Addresses

BSV addresses are like email addresses for money. They're derived from public keys and safe to share publicly.

### Creating Addresses

```typescript
// Method 1: From a private key (most common)
const privKey = bsv.PrivateKey.fromRandom()
const address = privKey.toAddress()
console.log('My address:', address.toString())
// Example: "1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm"

// Method 2: From a public key
const pubKey = privKey.toPublicKey()
const address2 = pubKey.toAddress()
console.log('Same address:', address2.toString())

// Method 3: From an address string
const address3 = bsv.Address.fromString('1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm')

// Method 4: Different networks
const testnetAddress = privKey.toAddress(bsv.Networks.testnet)
console.log('Testnet address:', testnetAddress.toString())
// Testnet addresses look different (start with 'm' or 'n')
```

### Address Types and Networks

```typescript
// BSV supports different address types
const p2pkhAddress = bsv.Address.fromPublicKey(pubKey)  // Pay to Public Key Hash (standard)
const p2shAddress = bsv.Address.fromScript(someScript)  // Pay to Script Hash

// Check address properties
console.log('Is P2PKH?', p2pkhAddress.isPayToPublicKeyHash())  // true
console.log('Network:', p2pkhAddress.network.name)              // "mainnet"

// Validate addresses
const isValid = bsv.Address.isValid('1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm')
console.log('Valid address?', isValid)  // true
```

## 🔐 Hash Functions

BSV uses various cryptographic hash functions. The `bsv` submodule provides easy access to all of them.

### Common Hash Functions

```typescript
import { bsv } from 'scrypt-ts'

// Prepare some data to hash
const data = Buffer.from('Hello BSV!', 'utf8')

// SHA-256 (most common in BSV)
const sha256Hash = bsv.crypto.Hash.sha256(data)
console.log('SHA-256:', sha256Hash.toString('hex'))
// Output: "b7e23ec29af22b7b2d5c2b01b2a2e6b5f5e5c5d5c5e5f5g5h5j5k5l5m5n5o5p5"

// Double SHA-256 (used for block hashes)
const hash256 = bsv.crypto.Hash.sha256sha256(data)
console.log('Double SHA-256:', hash256.toString('hex'))

// SHA-1 (legacy, avoid for new applications)
const sha1Hash = bsv.crypto.Hash.sha1(data)
console.log('SHA-1:', sha1Hash.toString('hex'))

// RIPEMD-160 (used in address generation)
const ripemd160Hash = bsv.crypto.Hash.ripemd160(data)
console.log('RIPEMD-160:', ripemd160Hash.toString('hex'))

// SHA-256 + RIPEMD-160 (used for P2PKH addresses)
const hash160 = bsv.crypto.Hash.sha256ripemd160(data)
console.log('HASH160:', hash160.toString('hex'))
```

### Practical Hash Examples

```typescript
// Example 1: Creating a hash puzzle contract
const secret = 'my-secret-value'
const secretHash = bsv.crypto.Hash.sha256(Buffer.from(secret, 'utf8'))

// In your sCrypt contract:
// @method()
// public unlock(preimage: ByteString) {
//     assert(sha256(preimage) == this.secretHash, 'Wrong secret!')
// }

// Example 2: Generating unique IDs
const timestamp = Date.now().toString()
const random = Math.random().toString()
const uniqueId = bsv.crypto.Hash.sha256(Buffer.from(timestamp + random, 'utf8'))
console.log('Unique ID:', uniqueId.toString('hex'))
```

## 🔨 Building Transactions

The transaction builder provides a flexible way to construct BSV transactions programmatically.

### Basic Transaction Structure

```typescript
// Create a new transaction
const tx = new bsv.Transaction()

// Add inputs (spending existing UTXOs)
tx.from({
    txId: 'a477af6b2667c29670467e4e0728b685ee07b240235771862318e29ddbe58458',
    outputIndex: 0,
    script: bsv.Script.fromASM('OP_DUP OP_HASH160 20 0x88d9931ea73d60eaf7e5671efc0552b912911f2a OP_EQUALVERIFY OP_CHECKSIG'),
    satoshis: 100000
})

// Add outputs (creating new UTXOs)
tx.to('1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm', 50000)  // Send 50k sats
tx.to('1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa', 30000)  // Send 30k sats

// Add change output (return leftover funds)
tx.change('1EHNa6Q4Jz2uvNExL497mE43ikXhwF6kZm')

// Sign the transaction
const privKey = bsv.PrivateKey.fromWIF('L4rK1yDtCWekvXuE6oXD9jCYfFNV2cWRpVuPLBcCU2z8TrnZQVUG')
tx.sign(privKey)

// The transaction is now ready to broadcast!
console.log('Transaction ID:', tx.id)
console.log('Raw transaction:', tx.toString())
```

### Advanced Transaction Features

```typescript
// Custom fee rate (satoshis per byte)
tx.feePerByte(1)  // 1 sat/byte is typical for BSV

// Add OP_RETURN data output
tx.addData('Hello from sCrypt!')

// Add custom script output
const customScript = bsv.Script.fromASM('OP_ADD OP_5 OP_EQUAL')
tx.addOutput(new bsv.Transaction.Output({
    script: customScript,
    satoshis: 1000
}))

// Get transaction details
console.log('Inputs:', tx.inputs.length)
console.log('Outputs:', tx.outputs.length)
console.log('Fee:', tx.getFee())
console.log('Size:', tx.toBuffer().length, 'bytes')
```

## ✍️ Signing Transactions

Transaction signing proves you have the right to spend the UTXOs.

### Basic Signing

```typescript
// Single signature
const privKey = bsv.PrivateKey.fromRandom()
const tx = new bsv.Transaction()
    .from(utxo)  // Add input
    .to(address, amount)  // Add output
    .change(changeAddress)  // Add change
    .sign(privKey)  // Sign all inputs

// The signing process:
// 1. Creates a signature hash (message to sign)
// 2. Signs with private key using ECDSA
// 3. Attaches signature to transaction input
```

### Custom Signing

```typescript
// Sign specific input with specific key
tx.sign(privKey, sigtype, inputIndex)

// Multiple signatures (different keys for different inputs)
tx.sign([privKey1, privKey2])

// Get signature for manual handling
const sig = bsv.Transaction.Sighash.sign(
    tx,
    privKey,
    sigtype,
    inputIndex,
    subscript,
    satoshis
)
```

## 📝 OP_RETURN Scripts

OP_RETURN outputs let you embed data in the blockchain.

```typescript
// Method 1: Simple data string
const tx1 = new bsv.Transaction()
tx1.addData('Hello BSV blockchain!')

// Method 2: Multiple data chunks
const tx2 = new bsv.Transaction()
tx2.addData(['Protocol', 'Version', '1.0'])

// Method 3: Binary data
const binaryData = Buffer.from([0x01, 0x02, 0x03, 0x04])
const tx3 = new bsv.Transaction()
tx3.addData(binaryData)

// Method 4: Build OP_RETURN script manually
const dataScript = bsv.Script.buildDataOut(['MyApp', 'Action', 'Parameters'])
tx.addOutput(new bsv.Transaction.Output({
    script: dataScript,
    satoshis: 0  // OP_RETURN outputs have 0 satoshis
}))
```

## 🔒 ECIES Encryption

Encrypt data using Elliptic Curve Integrated Encryption Scheme.

```typescript
// Encrypt message for a recipient
const recipientPubKey = bsv.PublicKey.fromHex('02b4632d...')
const message = 'Secret message for you!'

const encrypted = bsv.ECIES()
    .privateKey(senderPrivKey)
    .publicKey(recipientPubKey)
    .encrypt(message)

console.log('Encrypted:', encrypted.toString('base64'))

// Recipient decrypts with their private key
const decrypted = bsv.ECIES()
    .privateKey(recipientPrivKey)
    .publicKey(senderPubKey)
    .decrypt(encrypted)

console.log('Decrypted:', decrypted.toString('utf8'))
// Output: "Secret message for you!"
```

## 💡 Best Practices

### Security First
```typescript
// NEVER hardcode private keys
// BAD: const privKey = bsv.PrivateKey.fromWIF('L4rK1...')
// GOOD: const privKey = bsv.PrivateKey.fromWIF(process.env.PRIVATE_KEY)

// Always validate external input
if (!bsv.Address.isValid(userAddress)) {
    throw new Error('Invalid address provided')
}
```

### Network Awareness
```typescript
// Always specify network explicitly for production
const network = process.env.NETWORK === 'mainnet' 
    ? bsv.Networks.mainnet 
    : bsv.Networks.testnet

const privKey = bsv.PrivateKey.fromRandom(network)
const address = privKey.toAddress(network)
```

### Error Handling
```typescript
try {
    const tx = new bsv.Transaction()
        .from(utxo)
        .to(address, amount)
        .change(changeAddress)
        .sign(privKey)
} catch (error) {
    if (error.message.includes('Insufficient funds')) {
        console.error('Not enough BSV to create transaction')
    } else {
        console.error('Transaction creation failed:', error)
    }
}
```

## 🎯 Common Use Cases

### 1. Generate a New Wallet
```typescript
function createWallet(network = 'testnet') {
    const privKey = bsv.PrivateKey.fromRandom(
        network === 'mainnet' ? bsv.Networks.mainnet : bsv.Networks.testnet
    )
    
    return {
        privateKey: privKey.toWIF(),
        publicKey: privKey.toPublicKey().toHex(),
        address: privKey.toAddress().toString()
    }
}
```

### 2. Create a Hash Time-Locked Contract
```typescript
function createHTLC(secret: string, lockTime: number) {
    const secretHash = bsv.crypto.Hash.sha256(Buffer.from(secret, 'utf8'))
    
    // Build the locking script
    const script = bsv.Script()
        .add('OP_IF')
        .add(bsv.crypto.Hash.sha256(Buffer.from(secret)))
        .add('OP_EQUALVERIFY')
        .add('OP_DUP')
        .add('OP_HASH160')
        // ... rest of HTLC script
    
    return script
}
```

### 3. Verify a Message Signature
```typescript
function verifySignature(message: string, signature: string, pubKeyHex: string) {
    const pubKey = bsv.PublicKey.fromHex(pubKeyHex)
    const messageHash = bsv.crypto.Hash.sha256(Buffer.from(message, 'utf8'))
    
    return bsv.ECDSA.verify(messageHash, signature, pubKey)
}
```

## 📚 Next Steps

Now that you understand the BSV utilities:

1. **[Understanding UTXOs](./understanding-utxos.md)** - Deep dive into the UTXO model
2. **[Writing Your First Contract](../writing-contracts/basics.md)** - Put these tools to use
3. **[BSV Script Basics](./scripts-and-contracts.md)** - Connect to smart contracts

---

> 💡 **Remember**: The `bsv` submodule handles the complex cryptography so you can focus on building great applications!