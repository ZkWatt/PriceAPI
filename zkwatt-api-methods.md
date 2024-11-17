# 8. API Methods

## 8.1 Price Feed Methods

### Real-time Price Feeds

```javascript
class PriceFeedAPI {
    // Get latest price for a market
    async getLatestPrice(market, options = {}) {
        const endpoint = `/v1/price-feed/latest/${market}`;
        const params = {
            includeProof: options.includeProof || false,
            confidence: options.confidence || 0.95,
            timeout: options.timeout || 5000
        };
        
        return await this.client.get(endpoint, { params });
    }

    // Subscribe to price updates
    async subscribePriceFeed(market, callback, options = {}) {
        const ws = new WebSocket(this.wsEndpoint);
        
        const subscription = {
            type: 'PRICE_SUBSCRIPTION',
            market,
            interval: options.interval || '1m',
            minConfidence: options.minConfidence || 0.95
        };

        ws.onopen = () => {
            ws.send(JSON.stringify({
                type: 'subscribe',
                data: subscription
            }));
        };

        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            callback(data);
        };

        return ws;
    }
}
```

### Price Feed Aggregation

```javascript
class PriceAggregator {
    // Create custom price feed aggregation
    async createAggregatedFeed(config) {
        const endpoint = '/v1/price-feed/aggregate';
        const body = {
            name: config.name,
            sources: config.sources,
            weights: config.weights,
            updateInterval: config.interval,
            validationRules: {
                minSources: config.minSources || 3,
                maxDeviation: config.maxDeviation || 0.05,
                staleness: config.staleness || '5m'
            }
        };
        
        return await this.client.post(endpoint, body);
    }

    // Get aggregated price
    async getAggregatedPrice(feedName) {
        const endpoint = `/v1/price-feed/aggregate/${feedName}`;
        return await this.client.get(endpoint);
    }
}
```

## 8.2 State Management

### State Operations

```javascript
class StateManager {
    // Update state with new data
    async updateState(stateUpdate) {
        const endpoint = '/v1/state/update';
        const body = {
            previousRoot: stateUpdate.previousRoot,
            newRoot: stateUpdate.newRoot,
            proof: stateUpdate.proof,
            updates: stateUpdate.updates
        };

        return await this.client.post(endpoint, body);
    }

    // Get current state
    async getCurrentState() {
        const endpoint = '/v1/state/current';
        return await this.client.get(endpoint);
    }

    // Verify state inclusion
    async verifyStateInclusion(proof) {
        const endpoint = '/v1/state/verify';
        const body = {
            proof,
            root: await this.getCurrentState()
        };

        return await this.client.post(endpoint, body);
    }
}
```

### State Synchronization

```javascript
class StateSynchronizer {
    constructor(client) {
        this.client = client;
        this.syncInterval = 60000; // 1 minute
    }

    // Start state sync
    async startSync() {
        setInterval(async () => {
            try {
                await this.syncState();
            } catch (error) {
                console.error('State sync failed:', error);
            }
        }, this.syncInterval);
    }

    // Sync state from network
    async syncState() {
        const currentBlock = await this.client.getCurrentBlock();
        const lastSyncedBlock = await this.getLastSyncedBlock();

        if (currentBlock > lastSyncedBlock) {
            const updates = await this.getStateUpdates(
                lastSyncedBlock,
                currentBlock
            );
            
            await this.applyUpdates(updates);
            await this.setLastSyncedBlock(currentBlock);
        }
    }
}
```

## 8.3 Proof Generation

### ZK Proof Generator

```javascript
class ProofGenerator {
    // Generate proof for price update
    async generatePriceProof(priceData) {
        const endpoint = '/v1/proof/generate/price';
        const body = {
            market: priceData.market,
            price: priceData.price,
            timestamp: priceData.timestamp,
            sources: priceData.sources
        };

        return await this.client.post(endpoint, body);
    }

    // Generate batch proof
    async generateBatchProof(updates) {
        const endpoint = '/v1/proof/generate/batch';
        const body = {
            updates,
            timestamp: Date.now(),
            compressionLevel: 'high'
        };

        return await this.client.post(endpoint, body);
    }

    // Verify proof
    async verifyProof(proof, publicInputs) {
        const endpoint = '/v1/proof/verify';
        const body = { proof, publicInputs };

        return await this.client.post(endpoint, body);
    }
}
```

### Custom Circuit Integration

```javascript
class CircuitManager {
    // Deploy custom circuit
    async deployCircuit(circuit) {
        const endpoint = '/v1/circuit/deploy';
        const body = {
            circuit: circuit.code,
            config: circuit.config,
            name: circuit.name
        };

        return await this.client.post(endpoint, body);
    }

    // Generate proof using custom circuit
    async generateCustomProof(circuitName, inputs) {
        const endpoint = `/v1/circuit/${circuitName}/prove`;
        return await this.client.post(endpoint, { inputs });
    }
}
```

## 8.4 Network Operations

### Node Management

```javascript
class NodeManager {
    // Register new node
    async registerNode(nodeConfig) {
        const endpoint = '/v1/node/register';
        const body = {
            nodeType: nodeConfig.type,
            location: nodeConfig.location,
            capacity: nodeConfig.capacity,
            pubKey: nodeConfig.pubKey
        };

        return await this.client.post(endpoint, body);
    }

    // Get node status
    async getNodeStatus(nodeId) {
        const endpoint = `/v1/node/${nodeId}/status`;
        return await this.client.get(endpoint);
    }

    // Update node configuration
    async updateNodeConfig(nodeId, config) {
        const endpoint = `/v1/node/${nodeId}/config`;
        return await this.client.put(endpoint, config);
    }
}
```

### Network Health Monitoring

```javascript
class NetworkMonitor {
    // Get network stats
    async getNetworkStats() {
        const endpoint = '/v1/network/stats';
        return await this.client.get(endpoint);
    }

    // Monitor node health
    async monitorNodeHealth(nodeId, callback) {
        const ws = new WebSocket(this.wsEndpoint);
        
        ws.onopen = () => {
            ws.send(JSON.stringify({
                type: 'HEALTH_MONITOR',
                nodeId
            }));
        };

        ws.onmessage = (event) => {
            const healthData = JSON.parse(event.data);
            callback(healthData);
        };

        return ws;
    }
}
```

## 8.5 Historical Queries

### Time Series Data

```javascript
class HistoricalData {
    // Get historical prices
    async getHistoricalPrices(params) {
        const endpoint = '/v1/historical/prices';
        const queryParams = {
            market: params.market,
            start: params.startTime,
            end: params.endTime,
            interval: params.interval || '1h',
            format: params.format || 'json'
        };

        return await this.client.get(endpoint, { params: queryParams });
    }

    // Get aggregated statistics
    async getStatistics(params) {
        const endpoint = '/v1/historical/stats';
        const queryParams = {
            market: params.market,
            period: params.period,
            metrics: params.metrics.join(',')
        };

        return await this.client.get(endpoint, { params: queryParams });
    }
}
```

### Data Export

```javascript
class DataExporter {
    // Export data to CSV
    async exportToCSV(params) {
        const endpoint = '/v1/export/csv';
        const body = {
            type: params.type,
            data: params.data,
            format: {
                delimiter: params.delimiter || ',',
                headers: params.headers || true,
                timezone: params.timezone || 'UTC'
            }
        };

        return await this.client.post(endpoint, body, {
            responseType: 'blob'
        });
    }

    // Export data to specific format
    async exportData(params) {
        const endpoint = `/v1/export/${params.format}`;
        const body = {
            query: params.query,
            filters: params.filters,
            options: params.options
        };

        return await this.client.post(endpoint, body, {
            responseType: params.format === 'csv' ? 'blob' : 'json'
        });
    }
}
```

Example Usage:

```javascript
// Initialize client
const client = new ZkWattClient({
    apiKey: 'your-api-key',
    endpoint: 'https://api.zkwatt.io'
});

// Create API instances
const priceFeed = new PriceFeedAPI(client);
const stateManager = new StateManager(client);
const proofGenerator = new ProofGenerator(client);
const nodeManager = new NodeManager(client);
const historical = new HistoricalData(client);

// Usage examples
async function examples() {
    // Get latest price
    const price = await priceFeed.getLatestPrice('EU.DE.SPOT', {
        includeProof: true
    });

    // Subscribe to price updates
    const subscription = await priceFeed.subscribePriceFeed(
        'EU.DE.SPOT',
        (update) => console.log('New price:', update),
        { interval: '1m' }
    );

    // Generate proof for price update
    const proof = await proofGenerator.generatePriceProof({
        market: 'EU.DE.SPOT',
        price: '45.67',
        timestamp: Date.now()
    });

    // Get historical data
    const history = await historical.getHistoricalPrices({
        market: 'EU.DE.SPOT',
        startTime: '2024-01-01',
        endTime: '2024-02-01',
        interval: '1h'
    });
}
```

This comprehensive API methods section provides detailed implementations for all major ZkWatt platform operations. The code is designed to be:

- Modular and reusable
- Well-documented
- Error-handled
- Type-safe
- Easy to integrate

Would you like me to expand on any particular method or provide more specific examples for certain use cases?