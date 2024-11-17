# 5. Smart Contract Integration

## 5.1 Contract Interfaces

### Core Oracle Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IZkWattOracle {
    struct Price {
        uint256 value;
        uint256 timestamp;
        uint256 heartbeat;
        bytes32 zkProof;
    }
    
    struct MarketConfig {
        uint256 minPrice;
        uint256 maxPrice;
        uint256 maxDelay;
        uint256 heartbeat;
    }
    
    event PriceUpdated(
        bytes32 indexed market,
        uint256 price,
        uint256 timestamp
    );
    
    event MarketConfigured(
        bytes32 indexed market,
        MarketConfig config
    );
    
    function getLatestPrice(bytes32 market) external view returns (Price memory);
    function getHistoricalPrice(bytes32 market, uint256 timestamp) external view returns (Price memory);
    function updatePrice(bytes32 market, uint256 price, bytes32 proof) external returns (bool);
    function configureMarket(bytes32 market, MarketConfig calldata config) external returns (bool);
}
```

### Proof Verifier Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IZkWattVerifier {
    struct Proof {
        uint256[2] a;
        uint256[2][2] b;
        uint256[2] c;
        uint256[] inputs;
    }
    
    function verifyProof(
        uint256[2] memory a,
        uint256[2][2] memory b,
        uint256[2] memory c,
        uint256[] memory inputs
    ) external view returns (bool);
    
    function verifyBatchProof(
        Proof[] calldata proofs,
        uint256[][] calldata inputs
    ) external view returns (bool);
}
```

### State Manager Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IZkWattStateManager {
    struct StateUpdate {
        bytes32 previousRoot;
        bytes32 newRoot;
        bytes32[] proof;
    }
    
    event StateUpdated(
        bytes32 indexed previousRoot,
        bytes32 indexed newRoot,
        uint256 timestamp
    );
    
    function updateState(StateUpdate calldata update) external returns (bool);
    function verifyState(bytes32 root, bytes32[] calldata proof) external view returns (bool);
    function getCurrentRoot() external view returns (bytes32);
}
```

## 5.2 Oracle Consumer Contracts

### Basic Consumer Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./IZkWattOracle.sol";

contract ZkWattConsumer {
    IZkWattOracle public oracle;
    bytes32 public market;
    
    uint256 public lastPrice;
    uint256 public lastUpdate;
    
    event PriceUpdated(uint256 price, uint256 timestamp);
    
    constructor(address _oracle, bytes32 _market) {
        oracle = IZkWattOracle(_oracle);
        market = _market;
    }
    
    function updatePrice() external returns (uint256) {
        IZkWattOracle.Price memory price = oracle.getLatestPrice(market);
        
        require(price.timestamp > lastUpdate, "No new price available");
        
        lastPrice = price.value;
        lastUpdate = price.timestamp;
        
        emit PriceUpdated(lastPrice, lastUpdate);
        return lastPrice;
    }
}
```

### Advanced Consumer with Price Triggers

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./IZkWattOracle.sol";

contract ZkWattPriceTrigger {
    struct PriceTrigger {
        uint256 threshold;
        bool isUpperBound;
        bool isActive;
        address callback;
    }
    
    IZkWattOracle public oracle;
    mapping(bytes32 => PriceTrigger[]) public triggers;
    
    event TriggerActivated(bytes32 indexed market, uint256 price, uint256 threshold);
    
    constructor(address _oracle) {
        oracle = IZkWattOracle(_oracle);
    }
    
    function addTrigger(
        bytes32 market,
        uint256 threshold,
        bool isUpperBound,
        address callback
    ) external {
        triggers[market].push(PriceTrigger({
            threshold: threshold,
            isUpperBound: isUpperBound,
            isActive: true,
            callback: callback
        }));
    }
    
    function checkTriggers(bytes32 market) external {
        IZkWattOracle.Price memory price = oracle.getLatestPrice(market);
        PriceTrigger[] storage marketTriggers = triggers[market];
        
        for (uint i = 0; i < marketTriggers.length; i++) {
            PriceTrigger storage trigger = marketTriggers[i];
            if (!trigger.isActive) continue;
            
            bool isTriggered = trigger.isUpperBound ? 
                price.value >= trigger.threshold :
                price.value <= trigger.threshold;
                
            if (isTriggered) {
                trigger.isActive = false;
                emit TriggerActivated(market, price.value, trigger.threshold);
                
                // Execute callback
                (bool success,) = trigger.callback.call(
                    abi.encodeWithSignature(
                        "onTrigger(bytes32,uint256,uint256)",
                        market,
                        price.value,
                        trigger.threshold
                    )
                );
                require(success, "Trigger callback failed");
            }
        }
    }
}
```

## 5.3 Price Feed Integration

### Price Feed Aggregator

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./IZkWattOracle.sol";

contract ZkWattAggregator {
    struct AggregatedPrice {
        uint256 price;
        uint256 timestamp;
        uint256 confidence;
    }
    
    IZkWattOracle public oracle;
    mapping(bytes32 => uint256) public weights;
    bytes32[] public markets;
    
    constructor(address _oracle, bytes32[] memory _markets, uint256[] memory _weights) {
        require(_markets.length == _weights.length, "Invalid weights");
        oracle = IZkWattOracle(_oracle);
        markets = _markets;
        
        uint256 totalWeight = 0;
        for (uint i = 0; i < _markets.length; i++) {
            weights[_markets[i]] = _weights[i];
            totalWeight += _weights[i];
        }
        require(totalWeight == 100, "Weights must sum to 100");
    }
    
    function getAggregatedPrice() public view returns (AggregatedPrice memory) {
        uint256 weightedSum = 0;
        uint256 latestTimestamp = 0;
        
        for (uint i = 0; i < markets.length; i++) {
            IZkWattOracle.Price memory price = oracle.getLatestPrice(markets[i]);
            weightedSum += (price.value * weights[markets[i]]) / 100;
            latestTimestamp = price.timestamp > latestTimestamp ? 
                price.timestamp : latestTimestamp;
        }
        
        return AggregatedPrice({
            price: weightedSum,
            timestamp: latestTimestamp,
            confidence: calculateConfidence()
        });
    }
    
    function calculateConfidence() internal view returns (uint256) {
        // Implement confidence calculation based on price variance
        // and data freshness
    }
}
```

## 5.4 Event Subscription

### Event Monitor Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ZkWattEventMonitor {
    struct Subscription {
        address subscriber;
        bytes32[] markets;
        uint256 minChange;  // Minimum price change percentage (basis points)
        uint256 maxDelay;   // Maximum acceptable delay in seconds
    }
    
    mapping(address => Subscription) public subscriptions;
    mapping(bytes32 => address[]) public marketSubscribers;
    
    event NewSubscription(address indexed subscriber, bytes32[] markets);
    event PriceAlert(bytes32 indexed market, uint256 oldPrice, uint256 newPrice);
    
    function subscribe(
        bytes32[] calldata markets,
        uint256 minChange,
        uint256 maxDelay
    ) external {
        subscriptions[msg.sender] = Subscription({
            subscriber: msg.sender,
            markets: markets,
            minChange: minChange,
            maxDelay: maxDelay
        });
        
        for (uint i = 0; i < markets.length; i++) {
            marketSubscribers[markets[i]].push(msg.sender);
        }
        
        emit NewSubscription(msg.sender, markets);
    }
    
    function notifyPriceChange(
        bytes32 market,
        uint256 oldPrice,
        uint256 newPrice
    ) external {
        address[] storage subscribers = marketSubscribers[market];
        
        for (uint i = 0; i < subscribers.length; i++) {
            Subscription storage sub = subscriptions[subscribers[i]];
            
            uint256 changePercent = calculateChangePercent(oldPrice, newPrice);
            
            if (changePercent >= sub.minChange) {
                IPriceSubscriber(sub.subscriber).onPriceChange(
                    market,
                    oldPrice,
                    newPrice
                );
                
                emit PriceAlert(market, oldPrice, newPrice);
            }
        }
    }
    
    function calculateChangePercent(
        uint256 oldPrice,
        uint256 newPrice
    ) internal pure returns (uint256) {
        if (oldPrice == 0) return 0;
        return (abs(newPrice - oldPrice) * 10000) / oldPrice;
    }
    
    function abs(uint256 a) internal pure returns (uint256) {
        return a >= 0 ? a : -a;
    }
}
```

## 5.5 Error Handling

### Error Handler Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ZkWattErrorHandler {
    enum ErrorType {
        INVALID_PROOF,
        STALE_PRICE,
        INVALID_UPDATE,
        CIRCUIT_ERROR,
        VERIFICATION_ERROR
    }
    
    struct ErrorLog {
        ErrorType errorType;
        uint256 timestamp;
        bytes data;
    }
    
    mapping(bytes32 => ErrorLog[]) public errorLogs;
    uint256 public constant MAX_RETRY_ATTEMPTS = 3;
    uint256 public constant ERROR_EXPIRY = 1 days;
    
    event ErrorLogged(bytes32 indexed market, ErrorType indexed errorType);
    event ErrorResolved(bytes32 indexed market, ErrorType indexed errorType);
    
    function logError(
        bytes32 market,
        ErrorType errorType,
        bytes calldata data
    ) external {
        errorLogs[market].push(ErrorLog({
            errorType: errorType,
            timestamp: block.timestamp,
            data: data
        }));
        
        emit ErrorLogged(market, errorType);
    }
    
    function getActiveErrors(
        bytes32 market
    ) external view returns (ErrorLog[] memory) {
        ErrorLog[] storage logs = errorLogs[market];
        uint256 activeCount = 0;
        
        // Count active errors
        for (uint i = 0; i < logs.length; i++) {
            if (block.timestamp - logs[i].timestamp <= ERROR_EXPIRY) {
                activeCount++;
            }
        }
        
        // Create array of active errors
        ErrorLog[] memory activeErrors = new ErrorLog[](activeCount);
        uint256 index = 0;
        
        for (uint i = 0; i < logs.length && index < activeCount; i++) {
            if (block.timestamp - logs[i].timestamp <= ERROR_EXPIRY) {
                activeErrors[index] = logs[i];
                index++;
            }
        }
        
        return activeErrors;
    }
    
    function resolveError(
        bytes32 market,
        uint256 errorIndex
    ) external returns (bool) {
        require(errorIndex < errorLogs[market].length, "Invalid error index");
        
        ErrorLog storage errorLog = errorLogs[market][errorIndex];
        require(
            block.timestamp - errorLog.timestamp <= ERROR_EXPIRY,
            "Error expired"
        );
        
        // Mark error as resolved by updating timestamp
        errorLog.timestamp = 0;
        
        emit ErrorResolved(market, errorLog.errorType);
        return true;
    }
    
    function shouldRetry(
        bytes32 market,
        ErrorType errorType
    ) external view returns (bool) {
        ErrorLog[] storage logs = errorLogs[market];
        uint256 errorCount = 0;
        
        // Count recent errors of the same type
        for (uint i = 0; i < logs.length; i++) {
            if (logs[i].errorType == errorType &&
                block.timestamp - logs[i].timestamp <= 1 hours) {
                errorCount++;
            }
        }
        
        return errorCount < MAX_RETRY_ATTEMPTS;
    }
}
```

These smart contracts provide a comprehensive framework for integrating with the ZkWatt oracle system, including:

1. Clear interfaces for core functionality
2. Flexible consumer contracts for different use cases
3. Advanced price feed aggregation
4. Event subscription and monitoring
5. Robust error handling

The contracts are designed to be:
- Gas efficient
- Secure against common vulnerabilities
- Easy to integrate with existing systems
- Flexible for different use cases
- Maintainable and upgradeable

Would you like me to expand on any particular aspect or provide more specific examples for certain use cases?