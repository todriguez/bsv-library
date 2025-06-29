# The BSV Submodule

sCrypt exports a submodule named `bsv` which is an interface that helps you manage low-level things for the BSV blockchain, such as creating key pairs, building, signing and serializing BSV transactions, and more.

In the context of sCrypt, the `bsv` submodule is used primarily for managing key pairs and defining custom transaction builders, as demonstrated in the How to Write a Contract section.

The goal of this section is to guide you through the basics of using the `bsv` submodule.

## Importing

You can import the `bsv` submodule like this:

```typescript
import { bsv } from 'scrypt-ts'
```

## Private Keys

A `PrivateKey` object is basically a wrapper around a 256-bit integer.

You can generate a BSV private key (for `mainnet`) from a random value like this:

```typescript
// Generate a random private key for BSV mainnet
const privKey = bsv.PrivateKey.fromRandom()
// Same as: const privKey = bsv.PrivateKey.fromRandom(bsv.Network.mainnet)
```

To create a private key for the test network (also referred to as `testnet`), do the following instead:

```typescript
// Generate a random private key for BSV testnet
const privKey = bsv.PrivateKey.fromRandom(bsv.Networks.testnet)
```

The main difference between a mainnet and a testnet key is how they get serialized. Check out this [BSV Wiki page on WIFs](https://wiki.bitcoinsv.io/index.php/Wallet_Import_Format_(WIF)) which explains the differences in more detail.

You can also create `PrivateKey` objects from serialized keys like this:

```typescript
// Create private key from WIF (Wallet Import Format)
const privKey = bsv.PrivateKey.fromWIF('cVDFHtcTU1wn92AkvTyDbtVqyUJ1SFQTEEanAWJ288xvA7TEPDcZ')

// Create private key from hex string
const privKey2 = bsv.PrivateKey.fromString('e3a9863f4c43576cdc316986ba0343826c1e0140b0156263ba6f464260456fe8')
```

See the decimal value of the private key the following way:

```typescript
// Get the big number representation of the private key
console.log(privKey.bn.toString())
```

> **⚠️ Warning**  
> Private keys should be carefully stored and never be publicly revealed. Otherwise it may lead to loss of funds.

## Public Keys

A public key is derived from a private key and can be shared publicly. Mathematically, a public key is a point on the default elliptic curve that BSV uses, named [`SECP256K1`](https://wiki.bitcoinsv.io/index.php/Secp256k1). It is the curve's base point multiplied by the value of the private key.

You can get the public key corresponding to a private key the following way:

```typescript
// Generate a private key and derive its public key
const privKey = bsv.PrivateKey.fromRandom(bsv.Networks.testnet)
const pubKey = privKey.toPublicKey()
```

Similar to a private key, you can serialize and deserialize public keys:

```typescript
// Create public key from hex string
const pubKey = bsv.PublicKey.fromHex('03a687b08533e37d5a6ff5c8b54a9869d4def9bdc2a4bf8c3a5b3b34d8934ccd17')

// Serialize public key to hex string
console.log(pubKey.toHex())
// Output: 03a687b08533e37d5a6ff5c8b54a9869d4def9bdc2a4bf8c3a5b3b34d8934ccd17
```

## Addresses

You can get a BSV address from either the private key or the public key:

```typescript
// Generate a private key and its corresponding public key
const privKey = bsv.PrivateKey.fromRandom(bsv.Networks.testnet)
const pubKey = privKey.toPublicKey()

// Get address from private key
console.log(privKey.toAddress().toString())
// Example output: mxRjX2uxHHmS4rdSYcmCcp2G91eseb5PpF

// Get address from public key (same result)
console.log(pubKey.toAddress().toString())
// Example output: mxRjX2uxHHmS4rdSYcmCcp2G91eseb5PpF
```

Read this [BSV wiki page](https://wiki.bitcoinsv.io/index.php/Bitcoin_address) for more information on how BSV addresses are constructed.

## Hash Functions

The `bsv` submodule offers various hash functions that are commonly used in BSV. You can use them like so:

```typescript
// Calculate SHA256 hash of data
const hashString = bsv.crypto.Hash.sha256(Buffer.from('this is the data I want to hash')).toString('hex')
console.log(hashString)
// Output: f88eec7ecabf88f9a64c4100cac1e0c0c4581100492137d1b656ea626cad63e3
```

The hash functions available in the `bsv` submodule are:

| Hash Function | Output Length | Description |
|--------------|---------------|-------------|
| `sha256` | 32 bytes | The SHA256 hash. |
| `sha256sha256` | 32 bytes | The SHA256 hash of the SHA256 hash. Used for blocks and transactions. |
| `sha512` | 64 bytes | The SHA512 hash. Commonly used in applications. |
| `sha1` | 20 bytes | The SHA1 hash. |
| `ripemd160` | 20 bytes | The RIPEMD160 hash. |
| `sha256ripemd160` | 20 bytes | The RIPEMD160 hash of the SHA256 hash. Used in BSV addresses. |

Note however, that these bsv.js hash functions should not be confused with sCrypt's native hash functions. These functions cannot be used in a smart contract method.

## Constructing Transactions

The `bsv` submodule offers a flexible system for constructing BSV transactions. Users are able to define scripts, transaction inputs and outputs, and a whole transaction including its metadata. For a complete description of BSV's transaction format, please read the [BSV wiki page](https://wiki.bitcoinsv.io/index.php/Transaction).

As an exercise let's construct a simple P2PKH transaction from scratch and sign it.

> **Note**  
> As you will notice further in these docs, most of these steps won't be needed in a regular smart contract development workflow as sCrypt already does a lot of heavy lifting for you. This section serves more as a deeper look on what is happening under the hood.

You can create an empty transaction like this:

```typescript
// Create an empty BSV transaction
let tx = new bsv.Transaction()
```

Because the transaction will need an input that provides it with some funds, we can use the `from` function to add one that unlocks the specified UTXO:

```typescript
// Add an input to the transaction
tx.from({
    // TXID that contains the output you want to unlock:
    txId: 'f50b8c6dedea6a4371d17040a9e8d2ea73d369177737fb9f47177fbda7d4d387',
    // Index of the UTXO:
    outputIndex: 0,
    // Script of the UTXO. In this case it's a regular P2PKH script:
    script: bsv.Script.fromASM('OP_DUP OP_HASH160 fde69facc20be6eee5ebf5f0ae96444106a0053f OP_EQUALVERIFY OP_CHECKSIG').toHex(),
    // Value locked in the UTXO in satoshis:
    satoshis: 99904
})
```

Now, the transaction needs an output that will pay to the address `mxXPxaRvFE3178Cr6KK7nrQ76gxjvBQ4UQ` in our example:

```typescript
// Add an output to send funds to a specific address
tx.addOutput(
    new bsv.Transaction.Output({
        script: bsv.Script.buildPublicKeyHashOut('mxXPxaRvFE3178Cr6KK7nrQ76gxjvBQ4UQ'),
        satoshis: 99804,
    })
)
```

Notice how the output value is 100 satoshis less than the value of the UTXO we're unlocking. This difference is the transaction fee (sometimes also called the "miner fee"). The transaction fees are collected by miners when they mine a block, so adding a transaction fee basically acts as an incentive for miners to include your transaction in a block.

The amount of transaction fee you should pay depends on the fee rate and the bytes of the transaction. By adding an additional output to the transaction, we can control how much the transaction fee is actually paid. This output is called the **change output**. By adjusting the amount of change output, we can pay as little transaction fees as possible while meeting the needs of miners.

You can directly call the `change` function to add a change output to the transaction without calculating the change amount by yourself. This function is smart enough that it will only add the change output when the difference between all inputs and outputs is more than the required transaction fee.

```typescript
// Add a change output to return excess funds
tx.change('n4fTXc2kaKXHyaxmuH5FTKiJ8Tr4fCPHFy')
```

For the fee rate, you can also change it by calling `feePerKb`:

```typescript
// Set the fee rate to 50 satoshis per kilobyte
tx.feePerKb(50)
```

## Signing

Now that we have the transaction constructed, it's time to sign it. First, we need to seal the transaction, so it will be ready to sign. Then we call the `sign` function, which takes the private key that can unlock the UTXO we passed to the `from` function. In our example, this is the private key that corresponds to the address `n4fTXc2kaKXHyaxmuH5FTKiJ8Tr4fCPHFy`:

```typescript
// Seal and sign the transaction with the appropriate private key
tx = tx.seal().sign('cNSb8V7pRt6r5HrPTETq2Li2EWYEjA7EcQ1E8V2aGdd6UzN9EuMw')
```

Viola! That's it. This will add the necessary data to the transaction's input script: the signature and the public key of our signing key. Now our transaction is ready to be posted to the blockchain.

You can serialize the transaction like this:

```typescript
// Serialize the transaction to hex format for broadcasting
console.log(tx.serialize())
```

To broadcast a transaction, you can use any provider you like. For demo and test purposes you can paste the serialized transaction [here](https://whatsonchain.com/broadcast).

## OP_RETURN Scripts

If you want to post some arbitrary data on-chain, without any locking logic, you can use transaction outputs with an OP_RETURN script.

An example of an OP_RETURN script written in ASM format is this:

```
OP_FALSE OP_RETURN 734372797074
```

The opcodes `OP_FALSE OP_RETURN` will make the script unspendable. After them we can insert arbitrary chunks of data. The `734372797074` is actually the string `sCrypt` encoded as an `utf-8` hexadecimal string.

```javascript
// Convert string to hex
console.log(Buffer.from('sCrypt').toString('hex'))
// Output: 734372797074
```

An OP_RETURN script can also contain more than a single chunk of data:

```
OP_FALSE OP_RETURN 48656c6c6f 66726f6d 734372797074
```

The `bsv` submodule offers a convenient function to construct such scripts:

```typescript
// Build an OP_RETURN script with multiple data chunks
const opRetScript: bsv.Script = bsv.Script.buildSafeDataOut(['Hello', 'from', 'sCrypt'])
```

We can add the resulting `bsv.Script` object to an output as shown above.

## ECIES

ECIES (Elliptic Curve Integrated Encryption Scheme) is a hybrid encryption scheme that combines the strengths of public-key cryptography and symmetric encryption. It allows two parties, each having an elliptic curve key pair, to exchange encrypted messages. The `bsv` submodule provides the `ECIES` class to easily implement this encryption scheme in your sCrypt projects.

Here's how to use it:

### Encryption

To encrypt a message using ECIES:

1. First, create an instance of the `ECIES` class.
2. Specify the public key of the recipient with the `publicKey` method.
3. Call the `encrypt` method with the message you wish to encrypt.

```typescript
// Encrypt a message using ECIES
const msg = 'Hello sCrypt!'
const encryption = new bsv.ECIES()
encryption.publicKey(recipientPublicKey)
const ciphertext = encryption.encrypt(msg)
```

In this example, `recipientPublicKey` is the recipient's public key.

### Decryption

To decrypt a message:

1. Create another instance of the `ECIES` class.
2. Set the recipient's private key using the `privateKey` method.
3. Call the `decrypt` method, passing the ciphertext you wish to decrypt.

```typescript
// Decrypt a message using ECIES
const decryption = new bsv.ECIES()
decryption.privateKey(recipientPrivateKey)
const msg = decryption.decrypt(ciphertext)
console.log(msg)
// Output: "Hello sCrypt!"
```

In this example, `recipientPrivateKey` is the private key of the recipient (the one corresponding to the public key used for encryption).

## References

Take a look at the full `bsv` submodule reference for a full list of what functions it provides.

As the `bsv` submodule is based on MoneyButton's library implementation, take a look at their [video tutorial series](https://www.youtube.com/playlist?list=PLwj1dNv7vWsMKEaHGr8FF9d1vITfnL-Vn). Please note that some things might be slightly different as the videos are now at least 5 years old.