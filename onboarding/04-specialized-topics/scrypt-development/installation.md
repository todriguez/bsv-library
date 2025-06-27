# Installation Guide

## 🚀 Quick Start (5 minutes)

Get up and running with sCrypt in just a few minutes. This guide will walk you through setting up your development environment and creating your first BSV smart contract.

## Prerequisites

### 1. Node.js and NPM
You'll need Node.js version 16 or higher. Check your version:

```bash
# Check Node.js version
node --version  # Should show v16.x.x or higher

# Check NPM version
npm --version
```

If you need to install Node.js, download it from [nodejs.org](https://nodejs.org/en/download).

### 2. Git
Git is required for version control and project scaffolding:

```bash
# Check if Git is installed
git --version
```

If not installed, get it from [git-scm.com](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

### 3. Apple Silicon (M1/M2) Users
If you're on a Mac with Apple Silicon, ensure Rosetta is installed:

```bash
# Install Rosetta if needed
softwareupdate --install-rosetta --agree-to-license
```

## The sCrypt CLI Tool

The sCrypt CLI is your gateway to BSV smart contract development. It provides:

- **Project Scaffolding** - Best practice project structure
- **Compilation** - Convert sCrypt to Bitcoin Script
- **Testing Framework** - Built-in Mocha for unit tests
- **Code Quality** - Prettier formatting and ESLint
- **Publishing** - Share contracts via NPM

### Installation Options

#### Option 1: Use Without Installing (Recommended for First Time)

Try sCrypt immediately without global installation:

```bash
# Create a demo project to explore
npx scrypt-cli project my-first-contract

# Navigate to your new project
cd my-first-contract

# Install dependencies
npm install

# Run the example tests
npm test
```

#### Option 2: Global Installation

For regular development, install the CLI globally:

```bash
# Install globally
npm install -g scrypt-cli

# Verify installation
scrypt-cli --version

# Create a new project
scrypt-cli project my-first-contract
```

## Your First Project Structure

After creating a project, you'll see this structure:

```
my-first-contract/
├── contracts/           # Your sCrypt smart contracts
│   └── demo.ts         # Example contract
├── tests/              # Test files
│   └── demo.test.ts    # Example test
├── artifacts/          # Compiled contracts (generated)
├── .env                # Environment variables
├── package.json        # Project dependencies
├── tsconfig.json       # TypeScript configuration
└── scrypt.config.json  # sCrypt compiler settings
```

### Understanding Each Component

#### 📁 contracts/
This is where your sCrypt smart contracts live:

```typescript
// contracts/demo.ts
import { SmartContract, method, prop, assert } from 'scrypt-ts'

export class Demo extends SmartContract {
    // @prop decorator marks contract properties
    // These become part of the deployed contract state
    @prop()
    readonly x: bigint

    constructor(x: bigint) {
        super(...arguments)
        this.x = x
    }

    // @method decorator marks public contract methods
    // These compile to Bitcoin Script
    @method()
    public unlock(x: bigint) {
        // Assertions in sCrypt become script conditions
        assert(this.add(this.x, 1n) == x, 'incorrect sum')
    }

    @method()
    add(x: bigint, y: bigint): bigint {
        return x + y
    }
}
```

#### 🧪 tests/
Test files verify your contracts work correctly:

```typescript
// tests/demo.test.ts
import { expect } from 'chai'
import { Demo } from '../contracts/demo'
import { getDefaultSigner } from './utils/helper'

describe('Test SmartContract `Demo`', () => {
    before(async () => {
        await Demo.compile()
    })

    it('should pass the public method unit test successfully.', async () => {
        const demo = new Demo(1n)
        
        // Set up transaction context
        await demo.connect(getDefaultSigner())
        
        // Deploy the contract
        const deployTx = await demo.deploy()
        console.log('Demo contract deployed: ', deployTx.id)
        
        // Call the contract method
        const { tx: callTx } = await demo.methods.unlock(2n)
        console.log('Demo contract called: ', callTx.id)
    })
})
```

#### ⚙️ Configuration Files

**scrypt.config.json** - Compiler settings:
```json
{
  "compilerVersion": "latest",
  "sourceDir": "./contracts",
  "artifactDir": "./artifacts",
  "outputDir": "./scrypts"
}
```

**.env** - Environment variables:
```bash
# Network settings
NETWORK=testnet  # or 'mainnet' for production

# Private key for testing (NEVER commit real keys!)
PRIVATE_KEY=your_private_key_here
```

## Development Workflow

### 1. Write Your Contract

Create a new file in `contracts/`:

```typescript
// contracts/hashlock.ts
import { SmartContract, method, prop, assert, ByteString, sha256 } from 'scrypt-ts'

export class HashLock extends SmartContract {
    @prop()
    readonly hash: ByteString

    constructor(hash: ByteString) {
        super(...arguments)
        this.hash = hash
    }

    @method()
    public unlock(preimage: ByteString) {
        // Only unlock if the hash matches
        assert(sha256(preimage) == this.hash, 'Invalid preimage')
    }
}
```

### 2. Compile Your Contract

```bash
# Compile all contracts
npm run compile

# Watch mode - recompile on changes
npm run watch
```

This generates artifacts in the `artifacts/` directory.

### 3. Test Your Contract

```bash
# Run all tests
npm test

# Run specific test file
npm test tests/hashlock.test.ts

# Run with coverage
npm run test:coverage
```

### 4. Deploy to Network

```bash
# Deploy to testnet (default)
npm run deploy

# Deploy to mainnet (be careful!)
NETWORK=mainnet npm run deploy
```

## IDE Setup

### Visual Studio Code (Recommended)

Install these extensions for the best experience:

1. **sCrypt** - Syntax highlighting and IntelliSense
2. **TypeScript** - Full TypeScript support
3. **Prettier** - Code formatting
4. **ESLint** - Code quality

### Other IDEs

Any TypeScript-capable IDE works:
- WebStorm / IntelliJ IDEA
- Sublime Text (with TypeScript plugin)
- Atom (with TypeScript plugin)
- Vim/Neovim (with TypeScript LSP)

## Try in Browser

Don't want to install anything? Try sCrypt online:

🌐 **[Repl.it Demo](https://replit.com/@msinkec/scryptTS-demo)** - Fork and experiment instantly

## Common Installation Issues

### Issue: "command not found: scrypt-cli"

**Solution**: Use npx or ensure global npm binaries are in your PATH:
```bash
# Find npm global bin directory
npm config get prefix

# Add to your PATH (in ~/.bashrc or ~/.zshrc)
export PATH="$PATH:$(npm config get prefix)/bin"
```

### Issue: "Cannot find module 'scrypt-ts'"

**Solution**: Install dependencies in your project:
```bash
npm install
```

### Issue: Compilation errors on M1/M2 Mac

**Solution**: Ensure Rosetta is installed and try:
```bash
# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

## Next Steps

Now that you have sCrypt installed:

1. **[BSV Basics](./bsv-basics/README.md)** - Understand the fundamentals
2. **[Writing Your First Contract](./writing-contracts/basics.md)** - Step-by-step guide
3. **[Tutorial: Hello World](./tutorials/hello-world.md)** - Complete walkthrough

## Resources

- **[sCrypt GitHub](https://github.com/sCrypt-Inc/scryptTS)** - Source code and issues
- **[Discord Community](https://discord.gg/scrypt)** - Get help and share ideas
- **[Example Projects](https://github.com/sCrypt-Inc/boilerplate)** - Learn from real code

---

> 💡 **Pro Tip**: Start with the demo project and modify it. This is the fastest way to learn sCrypt!