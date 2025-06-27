# LARS - Local Automated Runtime System

## 🚀 Streamlined BSV Development Environment

LARS (Local Automated Runtime System) is a powerful CLI tool that simplifies running and managing BSV Overlay Services locally. It automates the creation and configuration of your development environment, making it seamless to build and test BSV applications.

## 📚 What is LARS?

LARS is your local development companion that:

- **Automates** Docker-based environment setup with MySQL, MongoDB, and ngrok
- **Manages** BSV Overlay Services using `@bsv/overlay-express`
- **Handles** both backend and frontend development environments
- **Hot-reloads** contracts and code changes automatically
- **Simplifies** key management for different networks (mainnet/testnet)
- **Bridges** to CARS (Cloud Automated Runtime System) for production deployment

## 🛠️ Prerequisites

Before using LARS, ensure you have:

1. **Node.js** - Version 20 or higher
2. **Docker** - With Docker Compose plugin
3. **ngrok** - For exposing local services
4. **MetaNet Client** - For funding Ninja keys
5. **Git** - For version control (optional)

## 📦 Installation

Install LARS as a development dependency:

```bash
# Install in your BSV project
npm install --save-dev @bsv/lars

# Or install globally
npm install -g @bsv/lars
```

## 🏗️ Project Structure

LARS expects a specific project structure:

```
your-bsv-project/
├── deployment-info.json    # Core configuration file
├── package.json
├── local-data/            # LARS-generated files
├── frontend/              # Frontend application
│   ├── package.json
│   ├── src/
│   └── public/
└── backend/               # Backend services
    ├── package.json
    ├── tsconfig.json
    ├── src/
    │   ├── topic-managers/
    │   ├── lookup-services/
    │   └── contracts/
    └── artifacts/
```

## 📋 Configuration with deployment-info.json

The heart of LARS configuration is the `deployment-info.json` file:

```json
{
  "schema": "bsv-app",
  "schemaVersion": "1.0",
  "topicManagers": {
    "tm_voting": "./backend/src/topic-managers/VotingTopicManager.ts",
    "tm_token": "./backend/src/topic-managers/TokenTopicManager.ts"
  },
  "lookupServices": {
    "ls_voting": {
      "serviceFactory": "./backend/src/lookup-services/VotingLookupServiceFactory.ts",
      "hydrateWith": "mongo"
    },
    "ls_token": {
      "serviceFactory": "./backend/src/lookup-services/TokenLookupServiceFactory.ts",
      "hydrateWith": "mongo"
    }
  },
  "frontend": {
    "language": "react",
    "sourceDirectory": "./frontend"
  },
  "contracts": {
    "language": "sCrypt",
    "baseDirectory": "./backend"
  },
  "configs": [
    {
      "name": "Local LARS",
      "network": "testnet",
      "provider": "LARS",
      "run": ["backend", "frontend"]
    }
  ]
}
```

### Configuration Options Explained

#### Topic Managers
```json
"topicManagers": {
  "tm_identifier": "path/to/TopicManager.ts"
}
```
Topic managers handle specific data types in your BSV application.

#### Lookup Services
```json
"lookupServices": {
  "ls_identifier": {
    "serviceFactory": "path/to/ServiceFactory.ts",
    "hydrateWith": "mongo" // or "mysql"
  }
}
```
Lookup services provide data retrieval and query capabilities.

#### Frontend Configuration
```json
"frontend": {
  "language": "react",    // or "static"
  "sourceDirectory": "./frontend"
}
```

#### Contract Settings
```json
"contracts": {
  "language": "sCrypt",
  "baseDirectory": "./backend"
}
```

#### LARS Configuration
```json
"configs": [{
  "name": "Local LARS",
  "network": "testnet",   // or "mainnet"
  "provider": "LARS",
  "run": ["backend"]      // or ["frontend"] or ["backend", "frontend"]
}]
```

## 🚀 Getting Started

### 1. Interactive Setup

Run LARS without arguments to access the interactive menu:

```bash
npx lars
```

The menu offers options to:
- Edit global keys
- Configure project settings
- Modify deployment info
- Start your environment

### 2. Key Management

LARS manages keys at two levels:

#### Global Keys (Reusable across projects)
Stored in `~/.lars-keys.json`:
```json
{
  "testnet": {
    "serverPrivateKey": "your-testnet-key",
    "taalApiKey": "your-testnet-taal-key"
  },
  "mainnet": {
    "serverPrivateKey": "your-mainnet-key",
    "taalApiKey": "your-mainnet-taal-key"
  }
}
```

#### Project-Level Keys
Stored in `local-data/lars-config.json` - overrides global keys for specific projects.

### 3. Starting Your Environment

From the menu or directly:

```bash
# Start with interactive menu
npx lars

# Or start directly
npx lars start
```

LARS will:
1. ✅ Check system dependencies
2. 🌐 Start ngrok tunnel
3. 🐳 Launch Docker containers (MySQL, MongoDB, OverlayExpress)
4. ⚛️ Start frontend if configured
5. 👀 Watch for code changes
6. 🔄 Auto-recompile sCrypt contracts

## 🔧 Advanced Features

### Hot Reloading

LARS watches your code and automatically:
- Recompiles sCrypt contracts when changed
- Restarts backend services
- Refreshes frontend (if using React)

```typescript
// Make changes to your contract
export class VotingContract extends SmartContract {
    @prop(true)
    votes: bigint  // LARS detects this change
    
    // ... contract logic
}
// Save file → LARS recompiles → Updates running service
```

### Network Switching

Easily switch between testnet and mainnet:

```bash
npx lars
# Select "Edit LARS Deployment Info"
# Choose "Change network"
# Select desired network
```

### Admin Tools

LARS provides admin functionality for your local OverlayExpress:

```bash
# Access admin tools
npx lars
# Select "Admin Tools Menu"
```

Options include:
- **Sync Advertisements** - Ensure local ads match configured topics
- **Start GASP Sync** - Trigger manual synchronization

### Advanced Engine Configuration

Configure OverlayExpress behavior:

```json
{
  "adminToken": "your-secure-token",
  "throwOnBroadcastFailure": false,
  "logTime": true,
  "logPrefix": "[MyApp]",
  "syncConfiguration": {
    "tm_voting": "https://peer.example.com/api",
    "tm_token": false  // Disable sync for this topic
  }
}
```

## 💡 Practical Examples

### Example 1: Setting Up a Token Project

```bash
# 1. Create project structure
mkdir my-token-project
cd my-token-project

# 2. Initialize project
npm init -y
npm install --save-dev @bsv/lars

# 3. Create deployment-info.json
cat > deployment-info.json << EOF
{
  "schema": "bsv-app",
  "schemaVersion": "1.0",
  "topicManagers": {
    "tm_token": "./backend/src/topic-managers/TokenTopicManager.ts"
  },
  "lookupServices": {
    "ls_token": {
      "serviceFactory": "./backend/src/lookup-services/TokenLookupServiceFactory.ts",
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
    "run": ["backend"]
  }]
}
EOF

# 4. Start LARS
npx lars
```

### Example 2: Adding Frontend to Existing Project

```bash
# 1. Update deployment-info.json
{
  "frontend": {
    "language": "react",
    "sourceDirectory": "./frontend"
  },
  "configs": [{
    "run": ["backend", "frontend"]  // Add frontend
  }]
}

# 2. Create React app
npx create-react-app frontend --template typescript

# 3. Start LARS - it will handle both services
npx lars start
```

### Example 3: Contract Development Workflow

```typescript
// backend/src/contracts/MyContract.ts
import { SmartContract, method, prop } from 'scrypt-ts'

export class MyContract extends SmartContract {
    @prop()
    readonly owner: PubKey
    
    @method()
    public unlock(sig: Sig) {
        assert(this.checkSig(sig, this.owner))
    }
}
```

```bash
# LARS automatically:
# 1. Detects the new contract
# 2. Compiles it to Bitcoin Script
# 3. Places artifacts in backend/artifacts/
# 4. Reloads services
```

## 🔍 Troubleshooting

### Port Conflicts

LARS uses these default ports:
- **8080** - OverlayExpress
- **3306** - MySQL
- **27017** - MongoDB
- **3000** - Frontend (React)

Kill conflicting processes:
```bash
# Find process using port
lsof -i :8080

# Kill process
kill -9 <PID>
```

### Docker Issues

```bash
# Ensure Docker is running
docker ps

# Clean up old containers
docker-compose -f local-data/docker-compose.yml down

# Remove volumes for fresh start
docker volume prune
```

### Key Funding

If your server key needs funding:
```bash
# LARS will prompt, or manually fund:
# 1. Get the address from LARS output
# 2. Send at least 10,000 satoshis
# 3. Wait for confirmation
```

## 🌟 Best Practices

### 1. **Version Control**
```bash
# Add to .gitignore
local-data/
.env
*.key
```

### 2. **Environment Separation**
```json
// Separate configs for different environments
"configs": [
  {
    "name": "Local Development",
    "network": "testnet",
    "provider": "LARS"
  },
  {
    "name": "Staging",
    "network": "testnet",
    "provider": "CARS"
  },
  {
    "name": "Production",
    "network": "mainnet",
    "provider": "CARS"
  }
]
```

### 3. **Contract Organization**
```
backend/src/contracts/
├── tokens/
│   ├── SimpleToken.ts
│   └── NFTToken.ts
├── governance/
│   └── VotingContract.ts
└── utilities/
    └── Escrow.ts
```

## 🚢 From LARS to CARS

LARS mirrors CARS (Cloud Automated Runtime System) structure, making deployment seamless:

```bash
# Local development with LARS
npx lars start

# When ready for cloud deployment
# Same deployment-info.json works with CARS!
cars deploy --config production
```

## 📚 Additional Resources

- **[BSV Overlay Services](https://github.com/bitcoin-sv/overlay-services)** - Core overlay documentation
- **[sCrypt Documentation](https://scrypt.io)** - Smart contract development
- **[CARS Documentation](https://github.com/bsv-blockchain/cars)** - Cloud deployment

## 🎯 Summary

LARS transforms BSV development by:
- **Automating** complex environment setup
- **Integrating** all necessary services
- **Enabling** rapid development cycles
- **Providing** seamless path to production

Start building with confidence knowing LARS handles the infrastructure while you focus on your application logic!

---

> 💡 **Pro Tip**: Use LARS for all local development, then seamlessly transition to CARS for production deployment using the same configuration!