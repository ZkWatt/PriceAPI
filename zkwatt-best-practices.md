# 11. Best Practices

## 11.1 Integration Patterns

### Repository Pattern

```typescript
interface IPriceFeedRepository {
    getLatest(market: string): Promise<PriceData>;
    getHistory(market: string, timeRange: TimeRange): Promise<PriceData[]>;
    subscribe(market: string, callback: PriceCallback): Subscription;
}

class PriceFeedRepository implements IPriceFeedRepository {
    private readonly client: ZkWattClient;
    private readonly cache: PriceCache;

    constructor(client: ZkWattClient) {
        this.client = client;
        this.cache = new PriceCache();
    }

    // Implementation with caching and retry logic
    async getLatest(market: string): Promise<PriceData> {
        try {
            // Check cache first
            const cached = this.cache.get(market);
            if (cached && !this.isStale(cached)) {
                return cached;
            }

            // Fetch fresh data
            const price = await this.client.priceFeed.getLatest(market);
            this.cache.set(market, price);
            return price;
        } catch (error) {
            throw new RepositoryError('Failed to fetch price', { cause: error });
        }
    }
}
```

### Factory Pattern for Proof Generation

```typescript
class ProofFactory {
    private readonly circuits: Map<string, Circuit>;

    // Create appropriate proof generator based on type
    createProofGenerator(type: ProofType): IProofGenerator {
        switch (type) {
            case ProofType.PRICE_UPDATE:
                return new PriceUpdateProofGenerator(this.circuits.get('price'));
            case ProofType.STATE_TRANSITION:
                return new StateTransitionProofGenerator(this.circuits.get('state'));
            case ProofType.BATCH:
                return new BatchProofGenerator(this.circuits.get('batch'));
            default:
                throw new Error(`Unsupported proof type: ${type}`);
        }
    }
}
```

### Observer Pattern for Price Updates

```typescript
class PriceUpdateObserver {
    private observers: Map<string, Set<PriceObserver>>;

    constructor() {
        this.observers = new Map();
    }

    // Subscribe to price updates
    subscribe(market: string, observer: PriceObserver): void {
        if (!this.observers.has(market)) {
            this.observers.set(market, new Set());
        }
        this.observers.get(market)!.add(observer);
    }

    // Notify all observers of price update
    notify(market: string, price: PriceData): void {
        const marketObservers = this.observers.get(market);
        if (marketObservers) {
            for (const observer of marketObservers) {
                observer.onPriceUpdate(price);
            }
        }
    }
}
```

## 11.2 Error Handling

### Error Classification and Recovery

```typescript
class ErrorHandler {
    private readonly retryPolicy: RetryPolicy;
    private readonly errorLogger: ErrorLogger;

    async handle<T>(operation: () => Promise<T>): Promise<T> {
        let attempts = 0;

        while (true) {
            try {
                return await operation();
            } catch (error) {
                attempts++;
                
                const classification = this.classifyError(error);
                const shouldRetry = this.shouldRetry(classification, attempts);

                if (!shouldRetry) {
                    throw this.enhanceError(error, classification);
                }

                await this.wait(this.calculateBackoff(attempts));
            }
        }
    }

    private classifyError(error: any): ErrorClassification {
        if (error instanceof NetworkError) {
            return {
                type: 'NETWORK',
                severity: 'MEDIUM',
                retryable: true
            };
        }
        if (error instanceof ProofVerificationError) {
            return {
                type: 'VERIFICATION',
                severity: 'HIGH',
                retryable: false
            };
        }
        // Add more classifications...
    }

    private calculateBackoff(attempt: number): number {
        return Math.min(1000 * Math.pow(2, attempt), 30000);
    }
}
```

### Circuit Breaker Implementation

```typescript
class CircuitBreaker {
    private failures: number = 0;
    private lastFailure: number = 0;
    private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';

    async execute<T>(operation: () => Promise<T>): Promise<T> {
        if (this.isOpen()) {
            throw new CircuitBreakerOpenError();
        }

        try {
            const result = await operation();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }

    private onSuccess(): void {
        this.failures = 0;
        this.state = 'CLOSED';
    }

    private onFailure(): void {
        this.failures++;
        this.lastFailure = Date.now();

        if (this.failures >= this.threshold) {
            this.state = 'OPEN';
        }
    }
}
```

## 11.3 Gas Optimization

### Batch Processing

```solidity
contract GasOptimizedOracle {
    struct BatchUpdate {
        bytes32[] markets;
        uint256[] prices;
        bytes32[] proofs;
    }

    // Batch update prices
    function batchUpdatePrices(BatchUpdate calldata update) external {
        require(
            update.markets.length == update.prices.length &&
            update.markets.length == update.proofs.length,
            "Invalid batch size"
        );

        for (uint256 i = 0; i < update.markets.length; i++) {
            // Use assembly for gas optimization
            assembly {
                // Store price
                mstore(0x00, update.markets[i])
                mstore(0x20, update.prices[i])
                sstore(keccak256(0x00, 0x40), 1)
            }

            emit PriceUpdated(
                update.markets[i],
                update.prices[i],
                block.timestamp
            );
        }
    }
}
```

### Storage Optimization

```solidity
contract StorageOptimized {
    // Pack multiple values into single storage slot
    struct PriceData {
        uint64 price;      // Plenty for price with 8 decimals
        uint32 timestamp;  // Good until year 2106
        uint32 confidence; // Confidence scaled to uint32
    }

    // Use bytes32 for fixed-size arrays
    mapping(bytes32 => PriceData) public prices;

    // Use events for historical data
    event PriceUpdated(
        bytes32 indexed market,
        uint64 price,
        uint32 timestamp
    );
}
```

## 11.4 Proof Generation

### Efficient Proof Generation

```typescript
class ProofGenerator {
    private readonly workerPool: WorkerPool;
    private readonly circuitCache: CircuitCache;

    constructor(config: ProofConfig) {
        this.workerPool = new WorkerPool(config.workers);
        this.circuitCache = new CircuitCache(config.cacheSize);
    }

    async generateProof(input: ProofInput): Promise<Proof> {
        // Get or compile circuit
        const circuit = await this.circuitCache.get(input.circuitType);

        // Generate witness
        const witness = await this.generateWitness(input, circuit);

        // Generate proof in parallel if possible
        const proof = await this.workerPool.generateProof(witness, circuit);

        return proof;
    }

    private async generateWitness(
        input: ProofInput,
        circuit: Circuit
    ): Promise<Witness> {
        // Optimize witness generation
        const optimizedInput = this.optimizeInput(input);
        return await circuit.generateWitness(optimizedInput);
    }
}
```

### Proof Batching Strategy

```typescript
class ProofBatcher {
    private readonly batchSize: number;
    private batch: ProofRequest[] = [];
    private timer: NodeJS.Timeout | null = null;

    constructor(batchSize: number = 10) {
        this.batchSize = batchSize;
    }

    async addProofRequest(request: ProofRequest): Promise<Proof> {
        this.batch.push(request);

        if (this.batch.length >= this.batchSize) {
            return this.processBatch();
        }

        // Start timer for partial batch
        if (!this.timer) {
            this.timer = setTimeout(() => this.processBatch(), 5000);
        }

        return new Promise((resolve) => {
            request.resolve = resolve;
        });
    }

    private async processBatch(): Promise<void> {
        if (this.timer) {
            clearTimeout(this.timer);
            this.timer = null;
        }

        const currentBatch = this.batch;
        this.batch = [];

        const proofs = await this.generateBatchProof(currentBatch);
        
        // Resolve individual promises
        currentBatch.forEach((request, index) => {
            request.resolve(proofs[index]);
        });
    }
}
```

## 11.5 Data Validation

### Input Validation Strategy

```typescript
class DataValidator {
    private readonly validators: Map<string, ValidatorFn>;
    private readonly schemas: Map<string, ValidationSchema>;

    constructor() {
        this.initializeValidators();
        this.initializeSchemas();
    }

    // Validate with custom rules and schema
    async validate(data: any, type: string): Promise<ValidationResult> {
        // Schema validation
        const schema = this.schemas.get(type);
        if (schema) {
            const schemaResult = await this.validateSchema(data, schema);
            if (!schemaResult.valid) {
                return schemaResult;
            }
        }

        // Custom validation
        const validator = this.validators.get(type);
        if (validator) {
            return await validator(data);
        }

        return { valid: true };
    }

    // Custom validators for specific types
    private initializeValidators() {
        this.validators.set('price', this.validatePrice.bind(this));
        this.validators.set('proof', this.validateProof.bind(this));
    }

    private async validatePrice(price: PriceData): Promise<ValidationResult> {
        // Check price bounds
        if (price.value <= 0 || price.value > 1000000) {
            return {
                valid: false,
                error: 'Price out of bounds'
            };
        }

        // Check timestamp
        if (price.timestamp > Date.now() + 5000) {
            return {
                valid: false,
                error: 'Future timestamp not allowed'
            };
        }

        // Check proof format
        if (!this.isValidProofFormat(price.proof)) {
            return {
                valid: false,
                error: 'Invalid proof format'
            };
        }

        return { valid: true };
    }
}
```

### Chain of Responsibility for Validation

```typescript
abstract class ValidationHandler {
    private nextHandler: ValidationHandler | null = null;

    setNext(handler: ValidationHandler): ValidationHandler {
        this.nextHandler = handler;
        return handler;
    }

    async handle(request: ValidationRequest): Promise<ValidationResult> {
        const result = await this.validate(request);
        
        if (!result.valid || !this.nextHandler) {
            return result;
        }

        return this.nextHandler.handle(request);
    }

    protected abstract validate(request: ValidationRequest): Promise<ValidationResult>;
}

// Format Validator
class FormatValidator extends ValidationHandler {
    protected async validate(request: ValidationRequest): Promise<ValidationResult> {
        // Validate data format
        if (!this.isValidFormat(request.data)) {
            return {
                valid: false,
                error: 'Invalid format'
            };
        }
        return { valid: true };
    }
}

// Range Validator
class RangeValidator extends ValidationHandler {
    protected async validate(request: ValidationRequest): Promise<ValidationResult> {
        // Validate value ranges
        if (!this.isInRange(request.data)) {
            return {
                valid: false,
                error: 'Value out of range'
            };
        }
        return { valid: true };
    }
}

// Usage
const validator = new FormatValidator();
validator
    .setNext(new RangeValidator())
    .setNext(new ProofValidator())
    .setNext(new ConsistencyValidator());

const result = await validator.handle(request);
```

Best Practices Summary:

1. Integration Patterns:
   - Use repository pattern for data access
   - Implement factory pattern for complex objects
   - Use observer pattern for real-time updates
   - Implement circuit breaker for external services

2. Error Handling:
   - Classify errors appropriately
   - Implement retry strategies
   - Use circuit breakers for failing services
   - Log errors with context

3. Gas Optimization:
   - Batch operations when possible
   - Optimize storage usage
   - Use events for historical data
   - Pack variables efficiently

4. Proof Generation:
   - Use worker pools for parallel processing
   - Implement proof batching
   - Cache frequently used circuits
   - Optimize witness generation

5. Data Validation:
   - Implement comprehensive validation strategy
   - Use chain of responsibility pattern
   - Validate at multiple levels
   - Maintain clear validation rules

Would you like me to expand on any of these topics or provide additional examples?