# Google Trends Analysis Component

## What is Search Trends Analysis and Why it Matters
Search volume analysis helps measure public interest in Solana. Rising search interest often precedes price movements because:
- New retail investors research before buying
- Increased searches during fear/greed events
- Regional trends can predict market moves

```javascript
class SearchTrendsAnalyzer {
    constructor(trendsData) {
        this.searchTerms = {
            primary: ['solana', 'sol crypto', 'sol price'],
            bullish: ['buy solana', 'solana wallet', 'how to buy sol'],
            bearish: ['sell solana', 'solana crash', 'sol price drop'],
            neutral: ['what is solana', 'solana news', 'solana blockchain']
        };
    }

    analyzeSearchTrends() {
        // Example data structure
        const exampleData = {
            'buy solana': {
                current: 85,    // Current search interest (0-100)
                previous: 60,   // Previous period
                average: 50     // 90-day average
            },
            'sell solana': {
                current: 35,
                previous: 45,
                average: 40
            }
            // ... more terms
        };

        // 1. Calculate interest scores by category
        const scores = {
            bullish: this.calculateCategoryScore(exampleData, this.searchTerms.bullish),
            bearish: this.calculateCategoryScore(exampleData, this.searchTerms.bearish),
            neutral: this.calculateCategoryScore(exampleData, this.searchTerms.neutral)
        };

        // Example calculation:
        // Bullish terms:
        // (85 - 50) / 50 = 0.70 (70% above average)
        // Bearish terms:
        // (35 - 40) / 40 = -0.125 (12.5% below average)

        // 2. Calculate momentum
        const momentum = this.calculateMomentum(exampleData);
        // (85 - 60) / 60 = 0.417 (41.7% increase)

        // 3. Combine into final score
        const trendScore = this.calculateFinalScore(scores, momentum);

        return {
            score: trendScore,
            details: this.generateDetails(scores, momentum)
        };
    }

    calculateCategoryScore(data, terms) {
        let totalScore = 0;
        let count = 0;

        terms.forEach(term => {
            if (data[term]) {
                // Calculate deviation from average
                const deviation = (data[term].current - data[term].average) / data[term].average;
                totalScore += deviation;
                count++;
            }
        });

        return count > 0 ? totalScore / count : 0;
    }

    calculateFinalScore(scores, momentum) {
        // Weight the components
        const weightedScore = (
            (scores.bullish * 0.4) +    // 0.70 * 0.4 = 0.28
            (scores.bearish * -0.4) +   // -0.125 * -0.4 = 0.05
            (momentum * 0.2)            // 0.417 * 0.2 = 0.0834
        );
        // = 0.28 + 0.05 + 0.0834 = 0.4134

        // Convert to 0-100 scale
        return (Math.tanh(weightedScore * 2) + 1) * 50;  // = 71.2
    }
}
```

# Whale Movement Analysis Component

## What are Whale Movements and Why They Matter
Whale movements track large holders' behavior. This matters because:
- Whales often have insider knowledge or deep market understanding
- Large movements can predict major price swings
- Accumulation/distribution patterns signal market direction

```javascript
class WhaleAnalyzer {
    constructor(blockchainData) {
        this.thresholds = {
            whale: 100000,     // 100k SOL minimum for whale status
            largeMove: 10000   // 10k SOL minimum for significant movement
        };
    }

    analyzeWhaleMovements() {
        // Example transaction data
        const exampleTx = {
            from: "WhaleWallet123",
            to: "Exchange456",
            amount: 25000,
            timestamp: "2024-03-15T10:30:00Z",
            type: "exchange_inflow" // or exchange_outflow
        };

        // 1. Analyze Exchange Flows
        const exchangeFlows = this.analyzeExchangeFlows();
        
        // 2. Analyze Whale Accumulation
        const accumulation = this.analyzeAccumulation();
        
        // 3. Calculate Wallet Concentration
        const concentration = this.analyzeConcentration();

        return this.calculateWhaleScore(exchangeFlows, accumulation, concentration);
    }

    analyzeExchangeFlows() {
        // Example 24h flow data
        const flows = {
            inflow: {
                sol: 150000,    // SOL flowing into exchanges
                usdc: 2500000   // USDC flowing into exchanges
            },
            outflow: {
                sol: 100000,    // SOL flowing out of exchanges
                usdc: 3500000   // USDC flowing out of exchanges
            }
        };

        // 1. Calculate net flows
        const netSolFlow = flows.inflow.sol - flows.outflow.sol;
        // = 150000 - 100000 = 50000 net inflow

        const netUsdcFlow = flows.inflow.usdc - flows.outflow.usdc;
        // = 2500000 - 3500000 = -1000000 net outflow

        // 2. Calculate flow ratio
        // Negative ratio means more stablecoins moving to exchanges (buying power)
        const flowRatio = netSolFlow / (netUsdcFlow / 30); // Normalize by SOL price
        // = 50000 / (-1000000 / 30) = -1.5

        return {
            flowRatio: flowRatio,
            netSolFlow: netSolFlow,
            netUsdcFlow: netUsdcFlow
        };
    }

    analyzeAccumulation() {
        // Example whale wallet changes
        const whaleChanges = [
            { wallet: "Whale1", change: 15000 },   // Accumulated
            { wallet: "Whale2", change: -8000 },   // Distributed
            { wallet: "Whale3", change: 25000 },   // Accumulated
            // ... more whales
        ];

        let totalAccumulation = 0;
        whaleChanges.forEach(whale => {
            totalAccumulation += whale.change;
        });

        // Total = 32000 SOL accumulated

        return {
            netChange: totalAccumulation,
            activeWhales: whaleChanges.length,
            accumulating: whaleChanges.filter(w => w.change > 0).length
        };
    }

    calculateWhaleScore(exchangeFlows, accumulation, concentration) {
        // 1. Score exchange flows (-1 to 1)
        const flowScore = -Math.tanh(exchangeFlows.flowRatio);
        // = -tanh(-1.5) = 0.905
        // Positive score because negative ratio indicates buying pressure

        // 2. Score accumulation (0 to 1)
        const accumulationScore = Math.tanh(accumulation.netChange / 100000);
        // = tanh(32000/100000) = 0.311

        // 3. Combine scores
        const rawScore = (
            (flowScore * 0.5) +          // 0.905 * 0.5 = 0.453
            (accumulationScore * 0.5)    // 0.311 * 0.5 = 0.156
        );
        // = 0.609

        // Convert to 0-100 scale
        return (rawScore + 1) * 50;  // = 80.45
    }
}
```

# Order Book Analysis Component

## What is Order Book Analysis and Why it Matters
Order book analysis shows us the immediate supply and demand picture. This matters because:
- Order imbalances predict short-term price movements
- Large orders show institutional interest levels
- Order book depth indicates market liquidity

```javascript
class OrderBookAnalyzer {
    constructor(orderBookData) {
        this.depthLevels = 100;  // How many price levels to analyze
        this.priceImpact = 0.02; // 2% price impact threshold
    }

    analyzeOrderBook(orderBook) {
        // Example order book data
        const exampleBook = {
            bids: [
                [30.5, 10000],  // [price, size]
                [30.4, 15000],
                [30.3, 20000]
            ],
            asks: [
                [30.6, 12000],
                [30.7, 18000],
                [30.8, 25000]
            ],
            currentPrice: 30.55
        };

        // 1. Calculate buy/sell pressure
        const pressure = this.calculatePressure(exampleBook);
        
        // 2. Analyze order book depth
        const depth = this.analyzeDepth(exampleBook);
        
        // 3. Detect large orders
        const whaleOrders = this.detectLargeOrders(exampleBook);

        return this.calculateBookScore(pressure, depth, whaleOrders);
    }

    calculatePressure(book) {
        let buyVolume = 0;
        let sellVolume = 0;
        const midPrice = book.currentPrice;

        // Calculate total volume within 2% of mid price
        book.bids.forEach(([price, size]) => {
            if (price >= midPrice * 0.98) {
                buyVolume += size;
            }
        });

        book.asks.forEach(([price, size]) => {
            if (price <= midPrice * 1.02) {
                sellVolume += size;
            }
        });

        // Example volumes:
        // buyVolume = 45000
        // sellVolume = 30000

        // Calculate pressure ratio
        const pressureRatio = buyVolume / sellVolume;
        // = 45000 / 30000 = 1.5

        return Math.tanh(pressureRatio - 1);
        // = tanh(0.5) = 0.462
    }

    calculateBookScore(pressure, depth, whaleOrders) {
        // Example calculations:
        // pressure = 0.462 (buy pressure)
        // depth = 0.8 (good liquidity)
        // whaleOrders = 0.3 (moderate large orders)

        const rawScore = (
            (pressure * 0.5) +     // 0.462 * 0.5 = 0.231
            (depth * 0.3) +        // 0.8 * 0.3 = 0.24
            (whaleOrders * 0.2)    // 0.3 * 0.2 = 0.06
        );
        // = 0.531

        // Convert to 0-100 scale
        return (rawScore + 1) * 50;  // = 76.55
    }
}
```

# Combining External Components

```javascript
class ExternalDataAnalyzer {
    constructor(allData) {
        this.socialAnalyzer = new SocialSentimentAnalyzer(allData.social);
        this.trendsAnalyzer = new SearchTrendsAnalyzer(allData.trends);
        this.whaleAnalyzer = new WhaleAnalyzer(allData.blockchain);
        this.bookAnalyzer = new OrderBookAnalyzer(allData.orderBook);
    }

    calculateExternalScore() {
        // Get individual scores
        const scores = {
            social: this.socialAnalyzer.calculateOverallSentiment(),     // 80.45
            trends: this.trendsAnalyzer.analyzeSearchTrends().score,    // 71.20
            whales: this.whaleAnalyzer.analyzeWhaleMovements(),         // 80.45
            orderBook: this.bookAnalyzer.analyzeOrderBook()             // 76.55
        };

        // Weight the components
        const weightedScore = (
            (scores.social * 0.25) +     // 80.45 * 0.25 = 20.11
            (scores.trends * 0.20) +     // 71.20 * 0.20 = 14.24
            (scores.whales * 0.30) +     // 80.45 * 0.30 = 24.14
            (scores.orderBook * 0.25)    // 76.55 * 0.25 = 19.14
        );
        // = 77.63

        return Math.round(weightedScore); // = 78
    }
}
```