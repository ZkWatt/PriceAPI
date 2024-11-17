# 14. Appendices

## 14.1 Market Codes

### Electricity Market Codes

```typescript
const ELECTRICITY_MARKETS = {
    // European Markets
    'EU.DE.SPOT': {
        description: 'German Electricity Spot Market',
        currency: 'EUR',
        unit: 'MWh',
        updateFrequency: '5m',
        minimumVolume: 0.1,
        priceDecimals: 2,
        operatingHours: '24/7'
    },
    'EU.FR.SPOT': {
        description: 'French Electricity Spot Market',
        currency: 'EUR',
        unit: 'MWh',
        updateFrequency: '5m',
        minimumVolume: 0.1,
        priceDecimals: 2,
        operatingHours: '24/7'
    },
    
    // US Markets
    'US.PJM.SPOT': {
        description: 'PJM Interconnection Spot Market',
        currency: 'USD',
        unit: 'MWh',
        updateFrequency: '5m',
        minimumVolume: 1.0,
        priceDecimals: 2,
        operatingHours: '24/7',
        zones: ['WEST', 'EAST', 'CENTRAL']
    },
    'US.ERCOT.SPOT': {
        description: 'Texas ERCOT Spot Market',
        currency: 'USD',
        unit: 'MWh',
        updateFrequency: '5m',
        minimumVolume: 1.0,
        priceDecimals: 2,
        operatingHours: '24/7'
    }
};
```

### Renewable Energy Certificates

```typescript
const REC_MARKETS = {
    // Solar RECs
    'REC.SOLAR.EU': {
        description: 'European Solar Renewable Energy Certificates',
        currency: 'EUR',
        unit: 'MWh',
        vintage: 'Current Year',
        validityPeriod: '12 months',
        minimumVolume: 1.0
    },
    'REC.SOLAR.US': {
        description: 'US Solar Renewable Energy Certificates',
        currency: 'USD',
        unit: 'MWh',
        vintage: 'Current Year',
        validityPeriod: '12 months',
        minimumVolume: 1.0,
        states: ['CA', 'NY', 'NJ']
    },
    
    // Wind RECs
    'REC.WIND.EU': {
        description: 'European Wind Renewable Energy Certificates',
        currency: 'EUR',
        unit: 'MWh',
        vintage: 'Current Year',
        validityPeriod: '12 months',
        minimumVolume: 1.0
    }
};
```

## 14.2 Error Reference

### Error Categories and Codes

```typescript
const ERROR_REFERENCE = {
    // Authentication Errors (1000-1999)
    AUTH: {
        1001: {
            code: 'INVALID_API_KEY',
            description: 'The provided API key is invalid or expired',
            solution: 'Verify your API key in the developer dashboard',
            severity: 'HIGH'
        },
        1002: {
            code: 'INSUFFICIENT_PERMISSIONS',
            description: 'Account lacks required permissions',
            solution: 'Upgrade account or request additional permissions',
            severity: 'HIGH'
        }
    },
    
    // Proof Generation Errors (2000-2999)
    PROOF: {
        2001: {
            code: 'PROOF_GENERATION_FAILED',
            description: 'Failed to generate zero-knowledge proof',
            solution: 'Check input parameters and system resources',
            severity: 'HIGH'
        },
        2002: {
            code: 'PROOF_VERIFICATION_FAILED',
            description: 'Proof verification failed on-chain',
            solution: 'Regenerate proof or check chain conditions',
            severity: 'HIGH'
        }
    },
    
    // Network Errors (3000-3999)
    NETWORK: {
        3001: {
            code: 'NODE_UNREACHABLE',
            description: 'Unable to reach specified node',
            solution: 'Check network connectivity and node status',
            severity: 'MEDIUM'
        },
        3002: {
            code: 'CONSENSUS_FAILURE',
            description: 'Failed to reach consensus',
            solution: 'Wait for network stability or retry',
            severity: 'HIGH'
        }
    }
};
```

## 14.3 Network Parameters

### Chain Configuration

```typescript
const NETWORK_PARAMETERS = {
    // Mainnet Configuration
    mainnet: {
        chainId: 1,
        networkName: 'ZkWatt Mainnet',
        rpcUrl: 'https://rpc.zkwatt.io',
        wsUrl: 'wss://ws.zkwatt.io',
        explorerUrl: 'https://explorer.zkwatt.io',
        contracts: {
            oracle: '0x1234...',
            validator: '0x5678...',
            bridge: '0x9abc...'
        },
        parameters: {
            blockTime: 15,
            maxGasLimit: 30000000,
            minStake: '100000000000000000000', // 100 tokens
            validatorCount: 100
        }
    },
    
    // Testnet Configuration
    testnet: {
        chainId: 5,
        networkName: 'ZkWatt Testnet',
        rpcUrl: 'https://testnet.rpc.zkwatt.io',
        wsUrl: 'wss://testnet.ws.zkwatt.io',
        explorerUrl: 'https://testnet.explorer.zkwatt.io',
        contracts: {
            oracle: '0xtest...',
            validator: '0xtest...',
            bridge: '0xtest...'
        },
        parameters: {
            blockTime: 5,
            maxGasLimit: 50000000,
            minStake: '1000000000000000000', // 1 token
            validatorCount: 10
        }
    }
};
```

### Protocol Parameters

```typescript
const PROTOCOL_PARAMETERS = {
    // Consensus Parameters
    consensus: {
        minValidators: 4,
        blockTime: 15000, // 15 seconds
        epochLength: 7200, // 30 hours
        minStake: '100000000000000000000', // 100 tokens
        slashingPenalty: '10000000000000000000', // 10 tokens
        validatorReward: '1000000000000000000', // 1 token per block
    },
    
    // Oracle Parameters
    oracle: {
        updateInterval: 300000, // 5 minutes
        maxDeviation: 0.1, // 10% max price deviation
        minSources: 3,
        maxDelay: 600000, // 10 minutes
        confidenceThreshold: 0.95
    },
    
    // Proof System Parameters
    proofSystem: {
        maxProofSize: 1024, // bytes
        maxPublicInputs: 10,
        maxConstraints: 1000000,
        curveType: 'bn254',
        securityLevel: 128 // bits
    }
};
```

## 14.4 Technical Specifications

### Hardware Requirements

```typescript
const HARDWARE_SPECIFICATIONS = {
    // Validator Node Requirements
    validator: {
        cpu: {
            minimum: {
                cores: 8,
                frequency: '3.2 GHz',
                architecture: 'x86_64'
            },
            recommended: {
                cores: 16,
                frequency: '3.5 GHz',
                architecture: 'x86_64'
            }
        },
        memory: {
            minimum: '16GB',
            recommended: '32GB',
            type: 'DDR4'
        },
        storage: {
            minimum: '500GB',
            recommended: '1TB',
            type: 'SSD',
            iops: '10000'
        },
        network: {
            minimum: '100 Mbps',
            recommended: '1 Gbps',
            latency: '<50ms'
        }
    },
    
    // Oracle Node Requirements
    oracle: {
        cpu: {
            minimum: {
                cores: 4,
                frequency: '2.8 GHz',
                architecture: 'x86_64'
            },
            recommended: {
                cores: 8,
                frequency: '3.2 GHz',
                architecture: 'x86_64'
            }
        },
        memory: {
            minimum: '8GB',
            recommended: '16GB',
            type: 'DDR4'
        },
        storage: {
            minimum: '250GB',
            recommended: '500GB',
            type: 'SSD',
            iops: '5000'
        }
    }
};
```

### Software Requirements

```typescript
const SOFTWARE_SPECIFICATIONS = {
    // Operating System Requirements
    operatingSystem: {
        linux: {
            distributions: ['Ubuntu 20.04 LTS', 'Ubuntu 22.04 LTS'],
            kernelVersion: '5.4 or higher',
            packages: [
                'build-essential',
                'cmake',
                'libssl-dev',
                'pkg-config'
            ]
        },
        memory: {
            swapSize: '8GB',
            hugePages: 'enabled'
        }
    },
    
    // Runtime Requirements
    runtime: {
        node: {
            version: '>=16.0.0',
            packages: [
                '@zkwatt/core',
                '@zkwatt/proof-generator',
                '@zkwatt/validator'
            ]
        },
        rust: {
            version: '>=1.65.0',
            features: ['nightly']
        },
        docker: {
            version: '>=20.10.0',
            compose: '>=2.0.0'
        }
    }
};
```

## 14.5 Glossary

### Technical Terms

```typescript
const GLOSSARY = {
    // Cryptographic Terms
    'Zero-Knowledge Proof': {
        definition: 'A method by which one party can prove to another party that a statement is true without revealing any information beyond the validity of the statement itself',
        context: 'Used in ZkWatt for private price feed validation',
        example: 'Proving price validity without revealing source data'
    },
    
    'Rollup': {
        definition: 'A scaling solution that performs transaction execution outside the main Ethereum chain but posts transaction data on layer 1',
        context: 'ZkWatt uses rollups for efficient price data aggregation',
        relatedTerms: ['Layer 2', 'Scaling']
    },
    
    // Market Terms
    'Price Feed': {
        definition: 'A continuous stream of price data for a specific market or asset',
        context: 'Real-time electricity price updates',
        example: 'EU.DE.SPOT price feed updates every 5 minutes'
    },
    
    'REC': {
        definition: 'Renewable Energy Certificate - A market-based instrument that represents the property rights to the environmental benefits of renewable electricity generation',
        context: 'Trading and verification of renewable energy credits',
        relatedTerms: ['Renewable Energy', 'Carbon Credits']
    },
    
    // Technical Infrastructure
    'Node': {
        definition: 'A participant in the ZkWatt network that maintains network state and/or validates transactions',
        types: ['Validator Node', 'Oracle Node', 'Relay Node'],
        responsibilities: [
            'State maintenance',
            'Transaction validation',
            'Price feed verification'
        ]
    },
    
    'Circuit': {
        definition: 'A mathematical representation of computational statements that can be proved using zero-knowledge proofs',
        context: 'Used for creating verifiable price feeds',
        components: [
            'Constraints',
            'Witness',
            'Public inputs',
            'Private inputs'
        ]
    }
};
```

### Market-Specific Terms

```typescript
const MARKET_GLOSSARY = {
    // Electricity Market Terms
    'Spot Market': {
        definition: 'Market for immediate delivery of electricity',
        examples: ['Day-ahead market', 'Intraday market'],
        relevance: 'Primary source of price feed data'
    },
    
    'Base Load': {
        definition: 'The minimum level of electricity demand over a period of 24 hours',
        context: 'Used in price calculations and demand forecasting',
        relatedTerms: ['Peak Load', 'Off-Peak']
    },
    
    // Price Feed Terms
    'Oracle': {
        definition: 'A system that provides external data to a blockchain network',
        context: 'ZkWatt oracle provides verified electricity price data',
        components: [
            'Data collection',
            'Validation',
            'Proof generation',
            'On-chain verification'
        ]
    },
    
    'Aggregation': {
        definition: 'The process of combining multiple price sources into a single verified price',
        methods: [
            'Time-weighted average',
            'Volume-weighted average',
            'Median calculation'
        ],
        properties: [
            'Manipulation resistance',
            'Accuracy',
            'Timeliness'
        ]
    }
};
```

This comprehensive appendices section provides:

1. Market Codes:
   - Detailed market specifications
   - Trading parameters
   - Market types and categories

2. Error Reference:
   - Complete error catalog
   - Solutions and severity levels
   - Error categories

3. Network Parameters:
   - Chain configurations
   - Protocol parameters
   - Network settings

4. Technical Specifications:
   - Hardware requirements
   - Software dependencies
   - System configurations

5. Glossary:
   - Technical terms
   - Market-specific terminology
   - Context and examples

Would you like me to expand on any particular section or provide additional details?

