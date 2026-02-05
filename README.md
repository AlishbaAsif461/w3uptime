# W3Uptime - Distributed Uptime Monitoring Platform

W3Uptime is a comprehensive, distributed uptime monitoring platform built on blockchain technology. It combines decentralized validators, real-time monitoring, incident management, and community governance to provide transparent and reliable website uptime tracking.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [System Components](#system-components)
- [Prerequisites](#prerequisites)
- [Environment Variables](#environment-variables)
- [Installation](#installation)
- [Development](#development)
- [Production Deployment](#production-deployment)
- [Database Setup](#database-setup)
- [Smart Contracts](#smart-contracts)
- [Validator Setup](#validator-setup)
- [API Documentation](#api-documentation)
- [Troubleshooting](#troubleshooting)

## Overview

W3Uptime is a decentralized monitoring solution that leverages a network of independent validators to continuously check website availability and performance. The platform integrates blockchain technology for transparency, reputation tracking, and community governance.

### Key Features

- Decentralized validator network with cryptographic verification
- Real-time uptime monitoring with global coverage
- Incident detection and escalation management
- Multi-channel alerting (Email, Slack, Webhooks)
- Public status pages with customizable branding
- Community governance with on-chain voting
- Reputation-based reward system
- Time-series data analytics with TimescaleDB
- AI-powered incident analysis assistant
- Blockchain integration for transparency and payments

## Architecture

W3Uptime follows a microservices architecture with the following components:

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend (Next.js)                       │
│  - User Dashboard                                                │
│  - Monitor Management                                            │
│  - Status Pages                                                  │
│  - Governance Interface                                          │
│  - Background Workers (Escalation, Blockchain Listeners)         │
└──────────────────┬──────────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┬──────────────────┐
        │                     │                  │
┌───────▼────────┐   ┌───────▼────────┐  ┌─────▼──────────┐
│      Hub       │   │ Data Ingestion │  │   TimescaleDB  │
│  - WebSocket   │   │  - Batch API   │  │  - Time-series │
│  - Validator   │   │  - Tick Storage│  │  - PostgreSQL  │
│    Management  │   │                │  │  - Hypertables │
│  - Monitor     │   │                │  │                │
│    Distribution│   │                │  │                │
└────────┬───────┘   └────────────────┘  └────────────────┘
         │
    ┌────▼────┐
    │Validator│ (Multiple instances)
    │  - CLI  │
    │  - HTTP │
    │  - Tick │
    │  - Sign │
    └─────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    External Services                             │
│  - Redis (Queue)                                                 │
│  - Ethereum (Smart Contracts)                                    │
│  - Email Service (Nodemailer)                                    │
│  - Slack API                                                     │
│  - AI Services (OpenAI, Anthropic)                               │
└─────────────────────────────────────────────────────────────────┘
```

### Data Flow

1. Hub distributes monitoring tasks to connected validators every 30 seconds
2. Validators perform HTTP checks and send signed results back to Hub
3. Hub validates signatures and batches results to Data Ingestion service
4. Data Ingestion stores monitor ticks in TimescaleDB
5. Frontend queries data for analytics and triggers escalations
6. Escalation workers process alerts through configured channels
7. Blockchain listeners sync on-chain events with database

## Technology Stack

### Frontend Application
- Next.js 15.4.4 with React 19
- TypeScript 5.8
- TailwindCSS 4.0 with Radix UI components
- TanStack Query for data fetching
- Recharts for analytics visualization
- Ethers.js 6.x for blockchain interaction
- BullMQ + Redis for background jobs
- Mapbox GL for geographic visualization

### Backend Services
- Node.js 20+ with Express 5.x
- WebSocket (ws) for real-time communication
- Prisma ORM for database operations
- TimescaleDB (PostgreSQL 17 with time-series extension)

### Infrastructure
- Docker & Docker Compose for containerization
- Turborepo for monorepo management
- Redis 7 for job queues and caching
- PgAdmin for database management

### Blockchain
- Ethereum (Sepolia testnet)
- Solidity 0.8.20
- OpenZeppelin contracts
- Smart contracts for governance and transactions

### Development Tools
- ESLint for code quality
- Prettier for code formatting
- TypeScript for type safety
- Nodemon for development hot-reload

## System Components

### 1. Frontend (`apps/frontend`)

Next.js application serving as the main user interface.

**Responsibilities:**
- User authentication with MetaMask wallet
- Monitor CRUD operations
- Incident timeline and management
- Status page builder
- Community governance interface
- Analytics dashboard
- Background workers (escalation, blockchain sync)

**Key Directories:**
- `app/` - Next.js 15 app router pages
- `components/` - React components
- `hooks/` - Custom React hooks
- `lib/` - Utility functions and services
- `workers/` - BullMQ background workers

**Port:** 8000

### 2. Hub Service (`apps/hub`)

WebSocket server managing validator connections and monitor distribution.

**Responsibilities:**
- Accept validator registrations via WebSocket
- Distribute monitoring tasks to validators (30-second intervals)
- Verify cryptographic signatures on validator responses
- Batch validated results to Data Ingestion service
- Track validator reputation (good/bad ticks)
- Maintain active validator connections

**Key Features:**
- Auto-reconnection with exponential backoff
- Message queuing during network interruptions
- Geographic location tracking per validator
- Session management with nonce/timestamp protection

**Port:** 8080

### 3. Data Ingestion Service (`apps/data-ingestion`)

RESTful API for high-throughput time-series data ingestion.

**Responsibilities:**
- Receive batched monitor tick data from Hub
- Insert data into TimescaleDB hypertables
- Validate request payloads
- Handle high-volume writes efficiently

**API Endpoints:**
- `POST /batch/ticks` - Batch insert monitor ticks
- `GET /ping` - Health check

**Port:** 4001

### 4. Validator (`apps/validator`)

CLI application that connects to Hub and performs uptime checks.

**Responsibilities:**
- Connect to Hub via WebSocket
- Receive monitoring tasks
- Perform HTTP/HTTPS requests
- Measure latency and status codes
- Sign results with private key
- Submit verified results to Hub

**Features:**
- Encrypted keystore with AES-128-CTR
- Automatic key derivation from private key
- Configurable session timeouts
- Paranoid mode for enhanced security
- Support for HTTP/HTTPS protocols
- SSL certificate validation

**Configuration:**
- JSON-based configuration files
- Environment variable support
- CLI-based settings management

### 5. Shared Packages (`packages/`)

#### `db` Package
- Prisma schema and client
- Database queries and utilities
- Migration scripts
- Seed data

**Models:**
- User, GeoLocation
- Monitor, MonitorTick (hypertable)
- Incident, Alert, TimelineEvent
- EscalationPolicy, EscalationLevel, EscalationLog
- StatusPage, Maintenance
- Proposal, ProposalVote, VoteCache
- SlackIntegration, Session

#### `common` Package
- Shared TypeScript types
- Smart contract ABIs
- Contract helper functions
- WebSocket message types

#### `ui` Package
- Shared React components

#### `eslint-config` Package
- Shared ESLint configurations

#### `typescript-config` Package
- Shared TypeScript configurations

## Prerequisites

### Required Software
- Node.js 18.0 or higher
- npm 10.2.4 or higher
- Docker 20.x or higher
- Docker Compose 2.x or higher
- Git

### Required Accounts
- MetaMask wallet (for blockchain interaction)
- Ethereum Sepolia testnet ETH (for contract deployment)
- Abstract API key (for geolocation)
- Slack workspace (optional, for Slack alerts)
- Email account with app password (for email alerts)
- OpenAI/Anthropic API key (optional, for AI assistant)

## Environment Variables

Create a `.env` file in the root directory with the following variables:

### Database Configuration
```bash
# PostgreSQL/TimescaleDB connection
DATABASE_URL="postgresql://postgres:password@localhost:5433/myapp"
POSTGRES_DB="myapp"
POSTGRES_USER="postgres"
POSTGRES_PASSWORD="password"
POSTGRES_PORT="5433"
```

### Redis Configuration
```bash
REDIS_HOST="localhost"
REDIS_PORT="6379"
```

### Application URLs
```bash
# Frontend application URL
NEXT_PUBLIC_URL="http://localhost:8000"

# Service URLs (for internal communication)
FRONTEND_URL="http://localhost:8000"
HUB_URL="http://localhost:8080"
DATA_INGESTION_URL="http://localhost:4001"
DATA_INGESTION_PORT="4001"
```

### Blockchain Configuration
```bash
# Ethereum RPC endpoint (Alchemy, Infura, or public RPC)
ETHEREUM_RPC_URL="https://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY"

# W3Governance contract address (deployed on Sepolia)
NEXT_PUBLIC_GOVERNANCE_CONTRACT_ADDRESS="0xYourContractAddress"

# Platform signer wallet (for reputation claims)
PLATFORM_SIGNER_PRIVATE_KEY="0xYourPrivateKey"
```

### Email Configuration
```bash
# Gmail SMTP configuration (requires app password)
GOOGLE_APP_USER="your-email@gmail.com"
GOOGLE_APP_PASSWORD="your-app-password"
```

### External APIs
```bash
# Abstract API for IP geolocation
ABSTRACTAPI_KEY="your-abstract-api-key"

# OpenAI for AI assistant (optional)
OPENAI_API_KEY="sk-your-openai-key"

# Anthropic for AI assistant (optional)
ANTHROPIC_API_KEY="sk-ant-your-anthropic-key"
```

### PgAdmin (Optional)
```bash
PGADMIN_EMAIL="admin@example.com"
PGADMIN_PASSWORD="admin"
PGADMIN_PORT="8080"
```

### Node Environment
```bash
NODE_ENV="development"  # or "production"
```

## Installation

### 1. Clone Repository
```bash
git clone https://github.com/your-org/w3uptime.git
cd w3uptime
```

### 2. Install Dependencies
```bash
npm install
```

This installs dependencies for all workspaces (apps and packages).

### 3. Setup Database
```bash
# Start TimescaleDB and Redis
npm run dev:services

# Generate Prisma client
cd packages/db
npx prisma generate

# Run database migrations
npx prisma migrate dev

# Seed database (optional)
npm run db:seed
```

### 4. Setup Environment Variables
```bash
# Copy example environment file
cp .env.example .env

# Edit .env with your configuration
nano .env
```

## Development

### Start All Services
```bash
# Start infrastructure (TimescaleDB, Redis, PgAdmin)
npm run dev:services

# In a new terminal, start all applications
npm run dev
```

This starts:
- Frontend: http://localhost:8000
- Hub: http://localhost:8080
- Data Ingestion: http://localhost:4001

### Start Individual Services
```bash
# Start only frontend
npm run dev --workspace=frontend

# Start only hub
npm run dev --workspace=hub

# Start only data-ingestion
npm run dev --workspace=data-ingestion

# Start validator CLI
cd apps/validator
npm run dev
```

### Database Management
```bash
# View database logs
npm run db:logs

# Open database shell
npm run db:shell

# Access PgAdmin
# Navigate to http://localhost:8080
# Login with PGADMIN_EMAIL and PGADMIN_PASSWORD

# Reset database (WARNING: deletes all data)
npm run db:reset

# Stop database
npm run db:stop
```

### Prisma Commands
```bash
# Generate Prisma client after schema changes
cd packages/db
npx prisma generate

# Create new migration
npx prisma migrate dev --name your_migration_name

# Apply migrations
npx prisma migrate deploy

# Open Prisma Studio (GUI for database)
npx prisma studio
```

### Build
```bash
# Build all apps and packages
npm run build

# Build specific workspace
npm run build --workspace=frontend
```

### Code Quality
```bash
# Run linting
npm run lint

# Format code with Prettier
npm run format

# Type check all TypeScript
npm run check-types
```

## Production Deployment

### 1. Build Docker Images

```bash
# Build all production images
docker build -f apps/frontend/dockerfile -t w3uptime-frontend:latest .
docker build -f apps/hub/dockerfile -t w3uptime-hub:latest .
docker build -f apps/data-ingestion/dockerfile -t w3uptime-data-ingestion:latest .
```

### 2. Deploy with Docker Compose

```bash
# Start production stack
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose -f docker-compose.prod.yml logs -f

# Stop production stack
docker-compose -f docker-compose.prod.yml down
```

### 3. Production Environment Variables

Ensure your production `.env` file contains:
- Secure database passwords
- Production RPC endpoints
- Valid API keys
- HTTPS URLs for public endpoints
- `NODE_ENV=production`

### 4. Database Migrations

```bash
# Run migrations in production
docker-compose -f docker-compose.prod.yml exec frontend npx prisma migrate deploy
```

### 5. Health Checks

Each service exposes health check endpoints:
- Frontend: `GET /api/health`
- Hub: `GET /health`
- Data Ingestion: `GET /ping`

## Database Setup

### TimescaleDB Hypertables

W3Uptime uses TimescaleDB for efficient time-series data storage.

**Hypertable:** `MonitorTick`
- Automatically partitioned by time
- Optimized for high-volume inserts
- Efficient time-based queries
- Automatic data retention policies (configurable)

**Indexes:**
- `createdAt` - Primary time index
- `monitorId + createdAt` - Per-monitor queries
- `status + createdAt` - Status-based aggregations
- `countryCode + createdAt` - Geographic analysis
- `continentCode + createdAt` - Continental analysis
- `city + createdAt` - City-level analysis

### Schema Overview

**User Management:**
- `User` - User accounts with wallet addresses
- `Session` - Active user sessions
- `GeoLocation` - Validator geographic data

**Monitoring:**
- `Monitor` - Website monitors
- `MonitorTick` - Time-series uptime data (hypertable)
- `Incident` - Detected outages
- `Alert` - Generated alerts

**Escalation:**
- `EscalationPolicy` - Alert routing rules
- `EscalationLevel` - Escalation stages
- `EscalationLog` - Escalation history
- `TimelineEvent` - Incident timeline

**Status Pages:**
- `StatusPage` - Public status pages
- `StatusPageSection` - Page sections
- `Update` - Status updates
- `Maintenance` - Scheduled maintenance

**Governance:**
- `Proposal` - Community proposals
- `ProposalVote` - Off-chain votes
- `VoteCache` - On-chain vote mirror
- `ProposalComment` - Discussion threads

**Integrations:**
- `SlackIntegration` - Slack workspace connections

**Blockchain:**
- `Transaction` - Deposit/withdrawal history

## Smart Contracts

### W3Governance Contract

**Network:** Ethereum Sepolia Testnet
**Compiler:** Solidity 0.8.20
**Dependencies:** OpenZeppelin 5.x

**Features:**
- On-chain proposal creation
- Direct voting (users pay gas)
- 2/3 majority required to pass
- Configurable voting periods (1-30 days)
- Proposal finalization
- Reputation-based access control
- Emergency pause capability

**Key Functions:**
- `createProposal(contentHash, votingDuration)` - Create proposal
- `vote(proposalId, support)` - Cast vote
- `finalizeProposal(proposalId)` - Finalize after voting
- `claimReputation(amount, nonce, expiry, signature)` - Claim off-chain reputation

**Events:**
- `ProposalCreated` - New proposal submitted
- `VoteCast` - Vote recorded
- `ProposalFinalized` - Voting completed
- `ReputationClaimed` - Reputation transferred on-chain

**Deployment:**
See `deployments/README.md` for detailed deployment instructions.

### UserTransactions Contract

**Purpose:** Handle user deposits and withdrawals

**Features:**
- Ether deposits and withdrawals
- Signature-based withdrawal authorization
- Configurable min/max withdrawal limits
- Nonce-based replay protection
- Emergency withdrawal by owner

## Validator Setup

### Initialize Validator

```bash
cd apps/validator
npm run build

# Initialize with private key
node build/main.js init \
  --wallet-name my-validator \
  --private-key 0xYourPrivateKey \
  --hub-url ws://localhost:8080
```

### Start Validator

```bash
# Start validator (default wallet)
node build/main.js start

# Start with specific wallet
node build/main.js start --wallet my-validator

# Start in paranoid mode (password required for each signing)
node build/main.js start --paranoid
```

### Validator Configuration

Configuration stored in `~/.w3uptime/config.json`

```json
{
  "hub": {
    "url": "ws://localhost:8080"
  },
  "monitoring": {
    "timeout": 30000,
    "concurrent": 5
  },
  "session": {
    "timeout": 3600000
  },
  "paranoid": false
}
```

### Validator Commands

```bash
# List wallets
node build/main.js list-wallets

# Configure hub URL
node build/main.js config set hub.url ws://your-hub:8080

# View configuration
node build/main.js config get

# Export configuration
node build/main.js config export

# Import configuration
node build/main.js config import --file config.json
```

## API Documentation

### Frontend API Routes

**Authentication:**
- `POST /api/auth/nonce` - Get authentication nonce
- `POST /api/auth/verify` - Verify signed message
- `POST /api/auth/logout` - Logout user

**Monitors:**
- `GET /api/monitors` - List monitors
- `POST /api/monitors` - Create monitor
- `PUT /api/monitors/[id]` - Update monitor
- `DELETE /api/monitors/[id]` - Delete monitor
- `GET /api/monitors/[id]/status` - Get monitor status

**Incidents:**
- `GET /api/incidents` - List incidents
- `GET /api/incidents/[id]` - Get incident details
- `POST /api/incidents/[id]/timeline` - Add timeline event
- `PUT /api/incidents/[id]/resolve` - Resolve incident

**Analytics:**
- `GET /api/analytics/uptime` - Uptime statistics
- `GET /api/analytics/latency` - Latency metrics
- `GET /api/analytics/geographic` - Geographic distribution

**Governance:**
- `GET /api/proposals` - List proposals
- `POST /api/proposals` - Create proposal
- `POST /api/proposals/[id]/vote` - Vote on proposal
- `GET /api/proposals/[id]/votes` - Get vote counts

**Blockchain:**
- `GET /api/blockchain/balance` - Get wallet balance
- `POST /api/blockchain/deposit` - Deposit funds
- `POST /api/blockchain/withdraw` - Request withdrawal
- `GET /api/deposits` - Get deposit history

**Status Pages:**
- `GET /api/public/status/[id]` - Public status page
- `GET /api/status-pages` - List user status pages
- `POST /api/status-pages` - Create status page
- `PUT /api/status-pages/[id]` - Update status page

### Hub WebSocket Protocol

**Client to Server:**

```typescript
// Registration
{
  type: "signup",
  signature: "0x...",
  data: {
    publicKey: "0x...",
    timestamp: 1234567890,
    nonce: "uuid"
  }
}

// Validation Result
{
  type: "validate",
  signature: "0x...",
  data: {
    publicKey: "0x...",
    callbackId: "uuid",
    status: "GOOD" | "BAD",
    latency: 123.45,
    timestamp: 1234567890
  }
}
```

**Server to Client:**

```typescript
// Validation Request
{
  type: "validate",
  data: {
    url: "https://example.com",
    callbackId: "uuid"
  }
}

// Error
{
  type: "error",
  data: {
    message: "Error description"
  }
}
```

### Data Ingestion API

**Batch Insert Ticks:**
```bash
POST /batch/ticks
Content-Type: application/json

{
  "ticks": [
    {
      "monitorId": "uuid",
      "validatorId": "uuid",
      "status": "GOOD",
      "latency": 123.45,
      "longitude": -122.4194,
      "latitude": 37.7749,
      "countryCode": "US",
      "continentCode": "NA",
      "city": "San Francisco",
      "createdAt": "2026-02-05T10:00:00Z"
    }
  ]
}
```

## Troubleshooting

### Common Issues

**1. Database Connection Errors**
```bash
# Check if TimescaleDB is running
docker ps | grep timescaledb

# Restart database
npm run db:reset

# Check connection string in .env
echo $DATABASE_URL
```

**2. Prisma Client Not Generated**
```bash
# Regenerate Prisma client
cd packages/db
npx prisma generate
```

**3. Port Already in Use**
```bash
# Kill process on port 8000
lsof -ti:8000 | xargs kill -9

# Or change port in package.json dev script
```

**4. Validator Connection Issues**
```bash
# Check Hub is running
curl http://localhost:8080/health

# Verify WebSocket URL in validator config
cat ~/.w3uptime/config.json
```

**5. Blockchain Listener Errors**
```bash
# Check RPC endpoint
curl -X POST $ETHEREUM_RPC_URL \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Verify contract address
echo $NEXT_PUBLIC_GOVERNANCE_CONTRACT_ADDRESS
```

**6. Redis Connection Errors**
```bash
# Check Redis is running
docker ps | grep redis

# Test Redis connection
redis-cli ping
```

**7. Email Sending Failures**
```bash
# Verify Gmail app password (not regular password)
# Enable 2FA and generate app password at:
# https://myaccount.google.com/apppasswords

# Test SMTP connection
telnet smtp.gmail.com 587
```

### Logs

**View Application Logs:**
```bash
# Frontend logs
docker-compose logs -f frontend

# Hub logs
docker-compose logs -f hub

# Data Ingestion logs
docker-compose logs -f data-ingestion

# Database logs
npm run db:logs
```

### Performance Issues

**High Database Load:**
- Check query performance with `EXPLAIN ANALYZE`
- Verify hypertable chunk intervals
- Review index usage
- Consider read replicas

**Slow API Responses:**
- Enable Next.js caching
- Use TanStack Query for client-side caching
- Implement Redis caching layer
- Optimize Prisma queries with `select`

**Memory Leaks:**
- Monitor with `docker stats`
- Check for unclosed database connections
- Review WebSocket connection cleanup
- Profile with Node.js inspector

### Development Tips

**Hot Reload Not Working:**
```bash
# Restart dev server with cache clear
rm -rf .next
npm run dev --workspace=frontend
```

**TypeScript Errors:**
```bash
# Check types across all workspaces
npm run check-types

# Restart TypeScript server in VS Code
# Cmd/Ctrl + Shift + P -> "TypeScript: Restart TS Server"
```

**Docker Build Failures:**
```bash
# Clear Docker cache
docker system prune -a

# Rebuild with no cache
docker build --no-cache -f apps/frontend/dockerfile -t w3uptime-frontend:latest .
```

## Additional Resources

- [TimescaleDB Documentation](https://docs.timescale.com/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Next.js Documentation](https://nextjs.org/docs)
- [Ethers.js Documentation](https://docs.ethers.org/v6/)
- [BullMQ Documentation](https://docs.bullmq.io/)
- [Turborepo Documentation](https://turbo.build/repo/docs)

## License

MIT

## Support

For issues and questions, please open a GitHub issue or contact the development team.

```
cd my-turborepo

# With [global `turbo`](https://turborepo.com/docs/getting-started/installation#global-installation) installed (recommended)
turbo login

# Without [global `turbo`](https://turborepo.com/docs/getting-started/installation#global-installation), use your package manager
npx turbo login
yarn exec turbo login
pnpm exec turbo login
```

This will authenticate the Turborepo CLI with your [Vercel account](https://vercel.com/docs/concepts/personal-accounts/overview).

Next, you can link your Turborepo to your Remote Cache by running the following command from the root of your Turborepo:

```
# With [global `turbo`](https://turborepo.com/docs/getting-started/installation#global-installation) installed (recommended)
turbo link

# Without [global `turbo`](https://turborepo.com/docs/getting-started/installation#global-installation), use your package manager
npx turbo link
yarn exec turbo link
pnpm exec turbo link
```

## On-Chain Governance

W3Uptime features a fully transparent on-chain governance system where all proposals and votes are recorded on the Sepolia Ethereum testnet. Users vote directly via MetaMask transactions, ensuring complete transparency and verifiability.

### Key Features

- **Direct On-Chain Voting**: Each vote is a blockchain transaction
- **Full Transparency**: All votes immediately visible on-chain
- **Real-Time Verification**: Users can verify their votes on Etherscan
- **No Trust Required**: Smart contract handles all vote counting

### Documentation

For detailed information about the governance system, see:
- [Governance Transparency Guide](./docs/governance-transparency.md)
- [Implementation Log](./docs/governance-feature.md)

### Quick Start

1. Connect MetaMask to Sepolia testnet
2. Get Sepolia ETH from faucet: https://sepoliafaucet.com
3. Navigate to Community > Governance
4. Create or vote on proposals

---

## Useful Links

Learn more about the power of Turborepo:

- [Tasks](https://turborepo.com/docs/crafting-your-repository/running-tasks)
- [Caching](https://turborepo.com/docs/crafting-your-repository/caching)
- [Remote Caching](https://turborepo.com/docs/core-concepts/remote-caching)
- [Filtering](https://turborepo.com/docs/crafting-your-repository/running-tasks#using-filters)
- [Configuration Options](https://turborepo.com/docs/reference/configuration)
- [CLI Usage](https://turborepo.com/docs/reference/command-line-reference)
