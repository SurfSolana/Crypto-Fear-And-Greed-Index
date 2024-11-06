# Trading Implementation Using the Fear & Greed Index

## Understanding Signal Strength and Position Sizing

```javascript
class FearGreedTrader {
    constructor(config = {}) {
        this.config = {
            maxPositionSize: 100,    // 100% = full position
            riskPerTrade: 2,         // 2% risk per trade
            minConfidence: 70,       // Minimum confidence score to trade
            extremeThreshold: 80,    // Threshold for extreme readings
            ...config
        };
    }

    calculatePositionSize(indexData) {
        const { score, analysis } = indexData;
        
        // Example calculation with real numbers:
        // Score: 78 (Extreme Greed)
        // Confidence: 85
        // Base volatility: 81.3

        // 1. Base Position Size from Score Extremity
        let baseSize = this.calculateBaseSize(score);
        
        // 2. Adjust for Confidence
        baseSize = this.adjustForConfidence(baseSize, analysis.confidence);
        
        // 3. Adjust for Volatility
        baseSize = this.adjustForVolatility(baseSize, analysis.components.internal.volatility);
        
        // 4. Apply Risk Limits
        return this.applyRiskLimits(baseSize);
    }

    calculateBaseSize(score) {
        // Larger positions near extremes, smaller in middle range
        if (score <= 20 || score >= 80) {
            // Maximum position size near extremes
            const extremity = Math.min(Math.abs(score - 50) / 50, 1);
            return this.config.maxPositionSize * extremity;
            // Example: score 78
            // extremity = |78 - 50| / 50 = 0.56
            // baseSize = 100 * 0.56 = 56
        } else {
            // Reduced size in non-extreme conditions
            return this.config.maxPositionSize * 0.5;
        }
    }

    adjustForConfidence(baseSize, confidence) {
        // Scale position by confidence
        const confidenceMultiplier = confidence.score / 100;
        // Example: 56 * (85/100) = 47.6
        return baseSize * confidenceMultiplier;
    }

    adjustForVolatility(baseSize, volatility) {
        // Reduce position size in high volatility
        const volatilityMultiplier = 1 - (volatility.score / 200);
        // Example: 47.6 * (1 - 81.3/200) = 31.3
        return baseSize * volatilityMultiplier;
    }

    applyRiskLimits(size) {
        // Ensure position size doesn't exceed risk parameters
        return Math.min(size, this.config.maxPositionSize);
    }

    generateEntryStrategy(indexData) {
        const { score, analysis } = indexData;
        const positionSize = this.calculatePositionSize(indexData);

        return {
            type: this.determineStrategyType(score),
            size: positionSize,
            entries: this.calculateEntryLevels(analysis),
            stops: this.calculateStopLevels(analysis),
            targets: this.calculateTargetLevels(analysis)
        };
    }

    determineStrategyType(score) {
        if (score <= 20) return 'OVERSOLD_ACCUMULATION';
        if (score <= 40) return 'CAUTIOUS_BUYING';
        if (score <= 60) return 'NEUTRAL_RANGING';
        if (score <= 80) return 'CAUTIOUS_PROFIT_TAKING';
        return 'OVERBOUGHT_DISTRIBUTION';
    }
}

// Trading Strategy Implementation
class TradingStrategy {
    constructor(indexData) {
        this.trader = new FearGreedTrader();
        this.indexData = indexData;
    }

    generateTradeSetup() {
        const score = this.indexData.score;  // 78
        const strategy = this.trader.generateEntryStrategy(this.indexData);

        // Example Real Trade Setup:
        return {
            setup: {
                sentiment: "Extreme Greed",
                action: this.getTradeAction(score),
                positionSize: strategy.size,  // 31.3% of max position
                entry: {
                    primary: "Current market price",
                    scaled: this.getScaledEntries(strategy)
                },
                stopLoss: this.calculateStopLoss(strategy),
                targets: this.calculateTargets(strategy)
            },
            rules: this.generateTradeRules(strategy)
        };
    }

    getTradeAction(score) {
        // Define specific actions based on score ranges
        if (score >= 80) {
            return {
                primary: "DISTRIBUTE",
                methods: [
                    "Scale out of existing longs",
                    "Set trailing stops on remaining position",
                    "Look for short entries on technical weakness"
                ]
            };
        } else if (score >= 65) {
            return {
                primary: "LIGHTEN",
                methods: [
                    "Take partial profits on strength",
                    "Raise stops to breakeven",
                    "Reduce new position sizes"
                ]
            };
        } else if (score <= 20) {
            return {
                primary: "ACCUMULATE",
                methods: [
                    "Scale into new longs",
                    "Average down on existing positions",
                    "Look for technical support levels"
                ]
            };
        } else if (score <= 35) {
            return {
                primary: "BUILD",
                methods: [
                    "Start new positions",
                    "Use tighter stops",
                    "Focus on strong technical setups"
                ]
            };
        } else {
            return {
                primary: "NEUTRAL",
                methods: [
                    "Trade shorter timeframes",
                    "Focus on range-bound strategies",
                    "Maintain balanced portfolio"
                ]
            };
        }
    }

    getScaledEntries(strategy) {
        // Example for distribution strategy (score 78)
        return [
            {
                percentage: 40,
                condition: "Immediate scaling",
                reason: "Extreme greed signal"
            },
            {
                percentage: 30,
                condition: "On first pullback",
                reason: "Technical retracement"
            },
            {
                percentage: 30,
                condition: "If confidence increases",
                reason: "Confirmation of trend"
            }
        ];
    }

    generateTradeRules(strategy) {
        // Example rules for high score (78)
        return {
            entry: [
                "Scale out in thirds on strength",
                "Use technical levels for precise exits",
                "Don't add to position above score 80"
            ],
            management: [
                "Trail stops using 3-day lows",
                "Take partial profits into strength",
                "Reduce position size if volatility increases"
            ],
            exit: [
                "Full exit if score exceeds 90",
                "Exit 50% if volume drops significantly",
                "Exit if whale accumulation turns negative"
            ]
        };
    }

    calculateRiskReward(strategy) {
        return {
            entryPrice: 100, // Example current price
            stops: {
                initial: 95,  // 5% initial stop
                breakeven: 103, // Move to breakeven at +3%
                trailing: "3-day low"
            },
            targets: {
                tier1: {
                    price: 110,
                    percentage: 40,
                    riskReward: 2
                },
                tier2: {
                    price: 120,
                    percentage: 40,
                    riskReward: 4
                },
                tier3: {
                    price: 135,
                    percentage: 20,
                    riskReward: 7
                }
            }
        };
    }
}

// Example Usage:
const indexData = {
    score: 78,
    analysis: {
        confidence: { score: 85 },
        components: {
            internal: {
                volatility: { score: 81.3 }
            }
        }
    }
};

const strategy = new TradingStrategy(indexData);
const tradeSetup = strategy.generateTradeSetup();

console.log('Trade Setup:', tradeSetup);
```

## Understanding the Trading Output

Example trade setup based on Fear & Greed score of 78:

```javascript
{
    setup: {
        sentiment: "Extreme Greed",
        action: {
            primary: "DISTRIBUTE",
            methods: [
                "Scale out of existing longs",
                "Set trailing stops on remaining position",
                "Look for short entries on technical weakness"
            ]
        },
        positionSize: 31.3,  // % of maximum position size
        entry: {
            primary: "Current market price",
            scaled: [
                {
                    percentage: 40,
                    condition: "Immediate scaling",
                    reason: "Extreme greed signal"
                },
                {
                    percentage: 30,
                    condition: "On first pullback",
                    reason: "Technical retracement"
                },
                {
                    percentage: 30,
                    condition: "If confidence increases",
                    reason: "Confirmation of trend"
                }
            ]
        },
        stopLoss: {
            initial: "5% below entry",
            breakeven: "+3% move",
            trailing: "3-day low"
        },
        targets: {
            tier1: { price: "+10%", size: "40% of position" },
            tier2: { price: "+20%", size: "40% of position" },
            tier3: { price: "+35%", size: "20% of position" }
        }
    },
    rules: {
        entry: [
            "Scale out in thirds on strength",
            "Use technical levels for precise exits",
            "Don't add to position above score 80"
        ],
        management: [
            "Trail stops using 3-day lows",
            "Take partial profits into strength",
            "Reduce position size if volatility increases"
        ],
        exit: [
            "Full exit if score exceeds 90",
            "Exit 50% if volume drops significantly",
            "Exit if whale accumulation turns negative"
        ]
    }
}
```
