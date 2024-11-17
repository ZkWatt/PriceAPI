# 7. Data Types and Formats

## 7.1 Request Methods

### HTTP Methods

ZkWatt API supports the following HTTP methods:

```typescript
enum RequestMethod {
  GET = 'GET',       // Retrieve data
  POST = 'POST',     // Submit data or trigger actions
  PUT = 'PUT',       // Update existing data
  DELETE = 'DELETE'  // Remove data or cancel subscriptions
}
```

### Standard Request Format

```typescript
interface ZkWattRequest {
  method: RequestMethod;
  endpoint: string;
  params?: RequestParams;
  body?: RequestBody;
  headers: RequestHeaders;
}

interface RequestParams {
  market?: string;
  timestamp?: number;
  interval?: string;
  limit?: number;
  format?: 'json' | 'xml' | 'csv';
}

interface RequestHeaders {
  'X-API-Key': string;
  'Content-Type': string;
  'Accept': string;
}
```

Example Requests:

```javascript
// Price feed request
const priceFeedRequest = {
  method: 'GET',
  endpoint: '/v1/price-feed/latest',
  params: {
    market: 'EU.DE.SPOT',
    interval: '5m'
  },
  headers: {
    'X-API-Key': 'your-api-key',
    'Accept': 'application/json'
  }
};

// Submit energy data
const energyDataRequest = {
  method: 'POST',
  endpoint: '/v1/energy-data/submit',
  body: {
    nodeId: '0x1234...',
    timestamp: Date.now(),
    production: 1500,
    consumption: 1200,
    proof: '0xabcd...'
  },
  headers: {
    'X-API-Key': 'your-api-key',
    'Content-Type': 'application/json'
  }
};
```

## 7.2 Response Formats

### Standard Response Structure

```typescript
interface ZkWattResponse<T> {
  success: boolean;
  data?: T;
  error?: ErrorResponse;
  timestamp: number;
  requestId: string;
  metadata: ResponseMetadata;
}

interface ResponseMetadata {
  version: string;
  latency: number;
  compression?: string;
  format: 'json' | 'xml' | 'csv';
  nodeId: string;
}

interface ErrorResponse {
  code: number;
  message: string;
  details?: any;
  path?: string;
}
```

Example Responses:

```json
// Successful price feed response
{
  "success": true,
  "data": {
    "market": "EU.DE.SPOT",
    "price": "45.67",
    "timestamp": 1678234567890,
    "confidence": 0.99,
    "proof": "0xabcd..."
  },
  "timestamp": 1678234567890,
  "requestId": "req_abc123",
  "metadata": {
    "version": "1.0.0",
    "latency": 45,
    "format": "json",
    "nodeId": "node_xyz789"
  }
}

// Error response
{
  "success": false,
  "error": {
    "code": 4003,
    "message": "Invalid proof format",
    "details": {
      "expectedFormat": "bytes32",
      "receivedFormat": "string"
    }
  },
  "timestamp": 1678234567890,
  "requestId": "req_def456",
  "metadata": {
    "version": "1.0.0",
    "latency": 12,
    "format": "json",
    "nodeId": "node_xyz789"
  }
}
```

## 7.3 Price Feed Formats

### Price Data Structure

```typescript
interface PriceFeed {
  market: string;
  price: string;
  timestamp: number;
  confidence: number;
  volume?: string;
  proof: string;
  metadata: PriceMetadata;
}

interface PriceMetadata {
  sources: number;
  updateInterval: string;
  lastUpdate: number;
  deviation: number;
  marketStatus: 'open' | 'closed' | 'halted';
}

interface PriceHistory {
  market: string;
  interval: string;
  prices: PricePoint[];
  statistics: PriceStatistics;
}

interface PricePoint {
  timestamp: number;
  price: string;
  volume?: string;
  proof: string;
}

interface PriceStatistics {
  high: string;
  low: string;
  average: string;
  standardDeviation: string;
  sampleCount: number;
}
```

Example Price Feed:

```json
{
  "market": "EU.DE.SPOT",
  "price": "52.34",
  "timestamp": 1678234567890,
  "confidence": 0.98,
  "volume": "1500000",
  "proof": "0xabcd...",
  "metadata": {
    "sources": 5,
    "updateInterval": "5m",
    "lastUpdate": 1678234567890,
    "deviation": 0.02,
    "marketStatus": "open"
  }
}
```

## 7.4 Proof Formats

### ZK Proof Structure

```typescript
interface ZkProof {
  protocol: 'groth16' | 'plonk' | 'stark';
  proof: ProofData;
  publicInputs: string[];
  auxiliaryData?: any;
}

interface ProofData {
  pi_a: string[];
  pi_b: string[][];
  pi_c: string[];
  protocol_version: string;
  curve: string;
}

interface VerificationKey {
  alpha_1: string[];
  beta_2: string[][];
  gamma_2: string[][];
  delta_2: string[][];
  ic: string[][];
}
```

Example Proof:

```json
{
  "protocol": "groth16",
  "proof": {
    "pi_a": [
      "0x2a14c238e149c11a6cf40602cb1f10e0eba31f0a32022cefe9fc6d4a2",
      "0x1f8945e12082ae3c11f0d4b7ee984b07bd2ee87455f2c59c02446c1a"
    ],
    "pi_b": [
      [
        "0x1d2f007ad3f2f4c057655277f6594a72c3bb538e509815385834357e",
        "0x2e0880c638939a3988b2b6ec1dd4c6d3bd58ea9852941564a10d46f4"
      ],
      [
        "0x27d4a70271c9f2f92768010c0b386ac9a41161226a8c48222a0c59e3",
        "0x1f2af117b90d47bea41499962cef9cea2b5fa6f974adc35b2ada741c"
      ]
    ],
    "pi_c": [
      "0x1cc79f91771a4695b86fcf1d73d9d4a143b0cd028027221c2a3942ae",
      "0x2c2a32fbc3d899804f7c4708f707f42c14451ef316e28b5594474687"
    ],
    "protocol_version": "1.0",
    "curve": "bn128"
  },
  "publicInputs": [
    "0x1234567890abcdef...",
    "0x9876543210fedcba..."
  ]
}
```

## 7.5 Error Codes

### Error Categories

```typescript
enum ErrorCategory {
  VALIDATION = 1000,    // Input validation errors
  AUTHENTICATION = 2000, // Auth and access errors
  PROOF = 3000,         // Proof generation/verification errors
  NETWORK = 4000,       // Network and connection errors
  DATA = 5000,          // Data processing errors
  NODE = 6000,          // Node operation errors
  CONSENSUS = 7000,     // Consensus-related errors
  SYSTEM = 8000         // System-level errors
}
```

### Detailed Error Codes

```typescript
const ERROR_CODES = {
  // Validation Errors (1000-1999)
  INVALID_MARKET: 1001,
  INVALID_TIMESTAMP: 1002,
  INVALID_INTERVAL: 1003,
  INVALID_PRICE: 1004,
  INVALID_VOLUME: 1005,
  
  // Authentication Errors (2000-2999)
  INVALID_API_KEY: 2001,
  EXPIRED_TOKEN: 2002,
  INSUFFICIENT_PERMISSIONS: 2003,
  RATE_LIMIT_EXCEEDED: 2004,
  
  // Proof Errors (3000-3999)
  INVALID_PROOF_FORMAT: 3001,
  PROOF_VERIFICATION_FAILED: 3002,
  PROOF_GENERATION_FAILED: 3003,
  INVALID_PUBLIC_INPUTS: 3004,
  
  // Network Errors (4000-4999)
  NODE_UNREACHABLE: 4001,
  CONSENSUS_TIMEOUT: 4002,
  NETWORK_CONGESTION: 4003,
  
  // Data Errors (5000-5999)
  DATA_NOT_FOUND: 5001,
  STALE_DATA: 5002,
  INCOMPLETE_DATA: 5003,
  DATA_CORRUPTION: 5004,
  
  // Node Errors (6000-6999)
  NODE_OFFLINE: 6001,
  INVALID_NODE_STATE: 6002,
  NODE_SYNC_FAILED: 6003,
  
  // Consensus Errors (7000-7999)
  CONSENSUS_FAILED: 7001,
  INVALID_VALIDATOR_SET: 7002,
  QUORUM_NOT_REACHED: 7003,
  
  // System Errors (8000-8999)
  INTERNAL_ERROR: 8001,
  MAINTENANCE_MODE: 8002,
  RESOURCE_EXHAUSTED: 8003
};
```

Error Response Examples:

```json
// Validation Error
{
  "success": false,
  "error": {
    "code": 1001,
    "message": "Invalid market identifier",
    "details": {
      "market": "EU.XX.SPOT",
      "validMarkets": ["EU.DE.SPOT", "EU.FR.SPOT"]
    }
  }
}

// Proof Error
{
  "success": false,
  "error": {
    "code": 3002,
    "message": "Proof verification failed",
    "details": {
      "proofId": "0x1234...",
      "reason": "Invalid public inputs",
      "verifierContract": "0xabcd..."
    }
  }
}

// Network Error
{
  "success": false,
  "error": {
    "code": 4002,
    "message": "Consensus timeout",
    "details": {
      "requiredValidators": 10,
      "receivedValidators": 7,
      "timeoutSeconds": 30
    }
  }
}
```

### Error Handling Best Practices

1. Always check the `success` field first
2. Handle error categories appropriately:
   - Validation errors (1000s): Review input data
   - Authentication errors (2000s): Check credentials
   - Proof errors (3000s): May require proof regeneration
   - Network errors (4000s): Implement retry logic
   - Data errors (5000s): Consider fallback data sources
   - Node errors (6000s): Check node status
   - Consensus errors (7000s): Wait and retry
   - System errors (8000s): Contact support

Example Error Handler:

```javascript
class ErrorHandler {
  static async handleError(error) {
    const errorCode = error.error?.code || 8001;
    
    switch (Math.floor(errorCode / 1000)) {
      case 1:
        return await this.handleValidationError(error);
      case 2:
        return await this.handleAuthError(error);
      case 3:
        return await this.handleProofError(error);
      case 4:
        return await this.handleNetworkError(error);
      case 5:
        return await this.handleDataError(error);
      case 6:
        return await this.handleNodeError(error);
      case 7:
        return await this.handleConsensusError(error);
      default:
        return await this.handleSystemError(error);
    }
  }

  static async handleProofError(error) {
    if (error.error.code === 3002) {
      // Attempt proof regeneration
      return await this.regenerateProof(error.details);
    }
    throw error;
  }

  static async handleNetworkError(error) {
    if (error.error.code === 4002) {
      // Implement exponential backoff
      return await this.retryWithBackoff(error.details);
    }
    throw error;
  }
}
```

This comprehensive data types and formats documentation ensures consistent data handling across the ZkWatt platform. Would you like me to elaborate on any particular aspect or provide more specific examples?