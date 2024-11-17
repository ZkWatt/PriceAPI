# 3. Oracle Price Feeds

## 3.1 Available Energy Markets

ZkWatt provides price feeds for major energy markets globally, organized by region and type.

### European Markets

```javascript
const EU_MARKETS = {
  // Electricity Spot Markets
  'EU.DE.SPOT': 'German Electricity Spot',
  'EU.FR.SPOT': 'French Electricity Spot',
  'EU.NL.SPOT': 'Dutch Electricity Spot',
  'EU.IT.SPOT': 'Italian Electricity Spot',
  
  // Electricity Futures
  'EU.DE.FUT': 'German Electricity Futures',
  'EU.FR.FUT': 'French Electricity Futures',
  
  // Natural Gas
  'EU.TTF.SPOT': 'Dutch TTF Gas Spot',
  'EU.NBP.SPOT': 'UK NBP Gas Spot'
};
```

Example usage:

```javascript
// Fetch German spot prices
const spotPrice = await client.priceFeed.getLatest({
  market: 'EU.DE.SPOT',
  interval: '1h'
});

// Subscribe to multiple markets
const subscription = client.priceFeed.subscribe([
  'EU.DE.SPOT',
  'EU.FR.SPOT',
  'EU.TTF.SPOT'
], {
  onUpdate: (price) => {
    console.log(`New price for ${price.market}: ${price.value}`);
  },
  interval: '5m'
});
```

### North American Markets

```javascript
const NA_MARKETS = {
  // ISO Markets
  'US.PJM.SPOT': 'PJM Interconnection',
  'US.ERCOT.SPOT': 'Texas ERCOT',
  'US.CAISO.SPOT': 'California ISO',
  
  // Natural Gas
  'US.HH.SPOT': 'Henry Hub Natural Gas',
  
  // Renewable Energy Credits
  'US.REC.SOLAR': 'Solar Renewable Credits',
  'US.REC.WIND': 'Wind Renewable Credits'
};
```

### Asian Markets

```javascript
const ASIA_MARKETS = {
  'AS.JP.SPOT': 'Japanese Electricity Spot',
  'AS.KR.SPOT': 'Korean Electricity Spot',
  'AS.SG.LNG': 'Singapore LNG Spot'
};
```

## 3.2 Price Feed Types

### Spot Price Feeds

Real-time market prices with sub-minute updates:

```javascript
// Basic spot price query
const spotPrice = await client.priceFeed.getSpot({
  market: 'EU.DE.SPOT',
  location: 'FRANKFURT'
});

// With additional parameters
const detailedSpot = await client.priceFeed.getSpot({
  market: 'EU.DE.SPOT',
  location: 'FRANKFURT',
  options: {
    includeVolume: true,
    includeBidAsk: true,
    includeProof: true
  }
});
```

### Futures Price Feeds

Forward curves and derivatives pricing:

```javascript
// Get futures curve
const futureCurve = await client.priceFeed.getFutureCurve({
  market: 'EU.DE.FUT',
  maturities: ['1M', '2M', '3M', '6M', '1Y'],
  baseDate: new Date()
});

// Subscribe to specific contract
const futuresSubscription = client.priceFeed.subscribeFutures({
  market: 'EU.DE.FUT',
  contract: 'DEC-2024',
  onUpdate: (price) => {
    console.log(`New futures price: ${price.value}`);
  }
});
```

### Renewable Energy Certificate (REC) Feeds

```javascript
// Get REC prices
const recPrices = await client.priceFeed.getREC({
  market: 'US.REC.SOLAR',
  vintage: '2024',
  state: 'CA'
});
```

### Custom Price Feeds

Create aggregated or derived price feeds:

```javascript
// Create custom spread feed
const spreadFeed = await client.priceFeed.createCustom({
  name: 'DE-FR-SPREAD',
  formula: '${EU.DE.SPOT} - ${EU.FR.SPOT}',
  updateInterval: '1m',
  validation: {
    minValue: -100,
    maxValue: 100,
    requireProof: true
  }
});
```

## 3.3 Update Frequency

### Standard Update Intervals

```javascript
const UPDATE_INTERVALS = {
  REAL_TIME: '30s',    // 30-second updates
  STANDARD: '5m',      // 5-minute updates
  HOURLY: '1h',        // Hourly updates
  DAILY: '1d'          // Daily updates
};

// Subscribe with specific interval
const subscription = client.priceFeed.subscribe('EU.DE.SPOT', {
  interval: UPDATE_INTERVALS.REAL_TIME,
  onUpdate: (price) => {
    console.log(`New price: ${price.value}`);
  }
});
```

### Custom Update Schedules

```javascript
// Create custom update schedule
const customSchedule = client.priceFeed.createSchedule({
  market: 'EU.DE.SPOT',
  schedule: [
    { time: '08:00', interval: '30s' },  // Market open - high frequency
    { time: '12:00', interval: '5m' },   // Mid-day - standard frequency
    { time: '17:00', interval: '30s' },  // Market close - high frequency
    { time: '18:00', interval: '1h' }    // After-hours - low frequency
  ],
  timezone: 'Europe/Berlin'
});
```

## 3.4 Historical Data Access

### Basic Historical Queries

```javascript
// Get historical prices
const historicalPrices = await client.priceFeed.getHistorical({
  market: 'EU.DE.SPOT',
  start: '2024-01-01',
  end: '2024-02-01',
  interval: '1h'
});

// Get specific timeframe with custom parameters
const customHistory = await client.priceFeed.getHistorical({
  market: 'EU.DE.SPOT',
  start: '2024-01-01T08:00:00Z',
  end: '2024-01-01T16:00:00Z',
  interval: '5m',
  options: {
    includeVolume: true,
    includeBidAsk: true,
    format: 'csv'
  }
});
```

### Batch Historical Queries

```javascript
// Batch request for multiple markets
const batchHistory = await client.priceFeed.getHistoricalBatch({
  markets: ['EU.DE.SPOT', 'EU.FR.SPOT', 'EU.TTF.SPOT'],
  start: '2024-01-01',
  end: '2024-02-01',
  interval: '1h'
});

// Export to CSV
await client.export.toCSV(batchHistory, 'price_history.csv');
```

### Time Series Analysis

```javascript
// Get statistical analysis
const analysis = await client.analytics.analyze({
  market: 'EU.DE.SPOT',
  period: '1M',
  metrics: [
    'mean',
    'standardDeviation',
    'minimum',
    'maximum',
    'volatility'
  ]
});

// Generate price duration curve
const durationCurve = await client.analytics.getDurationCurve({
  market: 'EU.DE.SPOT',
  period: '1Y',
  resolution: '1h'
});
```

## 3.5 Aggregation Methods

### Standard Aggregation Methods

```javascript
// Time-weighted average price (TWAP)
const twap = await client.aggregation.getTWAP({
  market: 'EU.DE.SPOT',
  window: '24h',
  interval: '5m'
});

// Volume-weighted average price (VWAP)
const vwap = await client.aggregation.getVWAP({
  market: 'EU.DE.SPOT',
  window: '24h',
  interval: '5m'
});

// Median price
const median = await client.aggregation.getMedian({
  market: 'EU.DE.SPOT',
  window: '24h',
  interval: '5m'
});
```

### Custom Aggregation Methods

```javascript
// Create custom aggregation method
const customAggregation = await client.aggregation.create({
  name: 'CUSTOM_WEIGHTED',
  markets: ['EU.DE.SPOT', 'EU.FR.SPOT'],
  weights: [0.7, 0.3],
  method: 'weighted_average',
  options: {
    minimumSources: 2,
    outlierRemoval: true,
    outlierDeviations: 2
  }
});

// Use custom aggregation
const aggregatedPrice = await client.aggregation.get({
  method: 'CUSTOM_WEIGHTED',
  timestamp: new Date()
});
```

### Proof Generation for Aggregated Prices

```javascript
// Generate proof for aggregated price
const proofedAggregation = await client.aggregation.getWithProof({
  method: 'CUSTOM_WEIGHTED',
  timestamp: new Date(),
  options: {
    includeInputs: true,
    generateCircuit: true
  }
});

// Verify aggregation proof on-chain
const verificationTx = await client.contracts.verifyAggregation(
  proofedAggregation.proof,
  proofedAggregation.publicInputs
);
```

### Error Handling and Fallback Methods

```javascript
// Create aggregation with fallback
const robustAggregation = await client.aggregation.createRobust({
  primary: {
    method: 'CUSTOM_WEIGHTED',
    markets: ['EU.DE.SPOT', 'EU.FR.SPOT'],
    weights: [0.7, 0.3]
  },
  fallback: [
    {
      method: 'TWAP',
      market: 'EU.DE.SPOT',
      window: '1h'
    },
    {
      method: 'LAST_VALID',
      maxAge: '1h'
    }
  ],
  validation: {
    minSources: 2,
    maxDeviation: 0.1,
    staleness: '5m'
  }
});
```

Each of these components can be combined to create sophisticated price feed systems that meet specific requirements for accuracy, reliability, and verifiability. The ZK proofs ensure that all aggregation steps can be verified on-chain while maintaining privacy and efficiency.
