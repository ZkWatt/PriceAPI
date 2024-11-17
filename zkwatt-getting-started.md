# 2. Getting Started

## 2.1 Quick Start Guide

### Installation

First, install the ZkWatt SDK using npm:

```bash
npm install @zkwatt/sdk
# or using yarn
yarn add @zkwatt/sdk
```

### Basic Setup

Create a new ZkWatt client instance:

```javascript
const { ZkWattClient } = require('@zkwatt/sdk');

const client = new ZkWattClient({
  network: 'mainnet', // or 'testnet'
  apiKey: 'your-api-key',
  providerUrl: 'https://eth-mainnet.provider.com'
});
```

### Quick Example

Fetch the latest electricity price feed:

```javascript
async function getLatestPrice() {
  try {
    const price = await client.priceFeed.getLatest({
      market: 'EU.DE.SPOT',  // German spot market
      interval: '1h'         // 1-hour interval
    });
    
    console.log('Latest Price:', price.value);
    console.log('Timestamp:', price.timestamp);
    console.log('Proof:', price.zkProof);
  } catch (error) {
    console.error('Error fetching price:', error);
  }
}
```

### Deploy Consumer Contract

```solidity
// PriceFeedConsumer.sol
pragma solidity ^0.8.0;

import "@zkwatt/contracts/interfaces/IZkWattConsumer.sol";

contract PriceFeedConsumer is IZkWattConsumer {
    address public zkWattOracle;
    
    constructor(address _oracle) {
        zkWattOracle = _oracle;
    }
    
    function getLatestPrice(bytes32 market) external view returns (uint256) {
        return IZkWattOracle(zkWattOracle).getLatestPrice(market);
    }
}
```

Deploy using truffle:

```bash
truffle migrate --network mainnet
```

## 2.2 Authentication

### API Key Generation

1. Visit the ZkWatt Developer Portal: https://developer.zkwatt.io
2. Register or login to your account
3. Navigate to API Keys section
4. Click "Generate New API Key"
5. Select required permissions:
   - Price Feed Access
   - Historical Data Access
   - Proof Generation
   - Node Operation

### Authentication Methods

#### REST API Authentication

Include your API key in the request headers:

```bash
curl -X GET "https://api.zkwatt.io/v1/price-feed/latest" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

#### WebSocket Authentication

```javascript
const ws = new WebSocket('wss://ws.zkwatt.io/v1');

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'auth',
    apiKey: 'YOUR_API_KEY'
  }));
};
```

#### SDK Authentication

```javascript
const client = new ZkWattClient({
  apiKey: 'YOUR_API_KEY',
  // Optional additional configuration
  authMethod: 'bearer',  // 'bearer' or 'basic'
  timeout: 5000,         // milliseconds
  retryAttempts: 3
});
```

## 2.3 Network Selection

ZkWatt operates on multiple networks to serve different needs:

### Available Networks

| Network | Description | Endpoint | Chain ID |
|---------|-------------|----------|-----------|
| Mainnet | Production network | https://mainnet.zkwatt.io | 1 |
| Testnet | Test network | https://testnet.zkwatt.io | 5 |
| Devnet  | Development network | https://devnet.zkwatt.io | 31337 |

### Network Configuration

#### SDK Configuration

```javascript
// Mainnet
const mainnetClient = new ZkWattClient({
  network: 'mainnet',
  apiKey: 'YOUR_API_KEY'
});

// Testnet
const testnetClient = new ZkWattClient({
  network: 'testnet',
  apiKey: 'YOUR_API_KEY'
});

// Custom RPC
const customClient = new ZkWattClient({
  network: 'custom',
  rpcUrl: 'YOUR_RPC_URL',
  apiKey: 'YOUR_API_KEY'
});
```

#### Smart Contract Addresses

```javascript
const NETWORK_ADDRESSES = {
  mainnet: {
    oracle: '0x1234...5678',
    stateManager: '0x8765...4321',
    proofVerifier: '0x9876...5432'
  },
  testnet: {
    oracle: '0xabcd...efgh',
    stateManager: '0xefgh...abcd',
    proofVerifier: '0xijkl...mnop'
  }
};
```

## 2.4 API Endpoints

### REST API Endpoints

#### Price Feeds

```bash
# Get latest price
GET /v1/price-feed/latest
curl -X GET "https://api.zkwatt.io/v1/price-feed/latest?market=EU.DE.SPOT" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Get historical prices
GET /v1/price-feed/historical
curl -X GET "https://api.zkwatt.io/v1/price-feed/historical?market=EU.DE.SPOT&from=1625097600&to=1625184000" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Subscribe to updates
POST /v1/price-feed/subscribe
curl -X POST "https://api.zkwatt.io/v1/price-feed/subscribe" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"market": "EU.DE.SPOT", "interval": "1h"}'
```

#### ZK Proofs

```bash
# Generate proof
POST /v1/proof/generate
curl -X POST "https://api.zkwatt.io/v1/proof/generate" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"type": "price_update", "data": {...}}'

# Verify proof
POST /v1/proof/verify
curl -X POST "https://api.zkwatt.io/v1/proof/verify" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"proof": "0x...", "publicInputs": [...]}'
```

### WebSocket API

```javascript
const ws = new WebSocket('wss://ws.zkwatt.io/v1');

// Subscribe to price updates
ws.send(JSON.stringify({
  type: 'subscribe',
  channel: 'price_feed',
  market: 'EU.DE.SPOT',
  interval: '1h'
}));

// Handle incoming messages
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('New price update:', data);
};
```

## 2.5 Rate Limits

### Standard Rate Limits

| Endpoint Type | Free Tier | Pro Tier | Enterprise Tier |
|---------------|-----------|-----------|-----------------|
| REST API | 60 req/min | 300 req/min | Custom |
| WebSocket | 10 conn/min | 50 conn/min | Custom |
| Proof Generation | 100 req/day | 1000 req/day | Custom |

### Rate Limit Headers

Response headers include rate limit information:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1625097600
```

### Handling Rate Limits

```javascript
class RateLimitHandler {
  constructor(client) {
    this.client = client;
    this.queue = [];
    this.processing = false;
  }

  async addRequest(request) {
    return new Promise((resolve, reject) => {
      this.queue.push({ request, resolve, reject });
      if (!this.processing) {
        this.processQueue();
      }
    });
  }

  async processQueue() {
    if (this.queue.length === 0) {
      this.processing = false;
      return;
    }

    this.processing = true;
    const { request, resolve, reject } = this.queue.shift();

    try {
      const response = await this.client.executeRequest(request);
      resolve(response);
      
      // Check rate limit headers
      const remaining = parseInt(response.headers['x-ratelimit-remaining']);
      if (remaining > 0) {
        this.processQueue();
      } else {
        const reset = parseInt(response.headers['x-ratelimit-reset']);
        const delay = (reset * 1000) - Date.now();
        setTimeout(() => this.processQueue(), delay);
      }
    } catch (error) {
      reject(error);
      this.processQueue();
    }
  }
}

// Usage
const handler = new RateLimitHandler(client);
handler.addRequest({
  method: 'GET',
  endpoint: '/price-feed/latest',
  params: { market: 'EU.DE.SPOT' }
}).then(response => {
  console.log('Price data:', response);
});
```

### Best Practices

1. Implement exponential backoff for retry logic
2. Cache frequently accessed data
3. Use WebSocket connections for real-time updates instead of polling
4. Batch requests when possible
5. Monitor rate limit headers and adjust request patterns accordingly

Example implementation of exponential backoff:

```javascript
async function fetchWithRetry(request, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await client.executeRequest(request);
    } catch (error) {
      if (error.status === 429) { // Rate limit exceeded
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

This comprehensive getting started guide provides developers with all the necessary information to begin integrating with the ZkWatt platform, including detailed code examples, best practices, and rate limit handling strategies.
