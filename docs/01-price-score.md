# Price Score Component Explained

## What it is and Why it Matters
The Price Score is a key component that measures market sentiment through price trends. Why? Because price trends at different time frames tell us different things about market psychology:
- Short-term trends (7 days) show us immediate market reaction and current sentiment
- Medium-term trends (25 days) show us developing market conviction
- Long-term trends (99 days) show us underlying market bias

## Breaking it Down with Simple Numbers

### 1. Moving Averages
Let's say we have these Solana prices for the last 7 days:
```javascript
const prices = [20, 22, 21, 24, 23, 25, 28];  // Current price is 28
```

#### Simple Moving Average (SMA) Calculation Example:
```javascript
// 7-day SMA = (20 + 22 + 21 + 24 + 23 + 25 + 28) ÷ 7 = 23.29

class TechnicalAnalysis {
    static calculateSMA(prices, period) {
        const sum = prices.reduce((a, b) => a + b, 0);
        return sum / period;
    }
}

// Usage:
const sma7 = TechnicalAnalysis.calculateSMA(prices, 7);  // = 23.29
```

Why this matters: If current price (28) is above SMA (23.29), it suggests bullish momentum. The further above, the more bullish the signal.

### 2. Price Position Score
```javascript
class PriceScoreCalculator {
    calculatePricePosition(currentPrice, movingAverage) {
        return (currentPrice - movingAverage) / movingAverage;
    }
}

// Example:
const currentPrice = 28;
const sma = 23.29;
const position = (28 - 23.29) / 23.29;  // = 0.202 or 20.2% above MA
```

Real-world meaning: 
- Position = 0.202 means price is 20.2% above average
- Positive values suggest greed (buyers in control)
- Negative values suggest fear (sellers in control)

### 3. Trend Strength
Let's say we have two consecutive days of moving averages:
```javascript
const yesterdayMA = 22.50;
const todayMA = 23.29;

const trendStrength = (todayMA - yesterdayMA) / yesterdayMA;
// = (23.29 - 22.50) / 22.50
// = 0.035 or 3.5% increase
```

Why this matters:
- Positive trend strength means averages are rising (building momentum)
- Negative trend strength means averages are falling (losing momentum)
- Larger values indicate stronger trends

### Complete Implementation with Examples

```javascript
class PriceScoreCalculator {
    constructor(prices) {
        this.prices = prices;
        // We use different periods to capture different market psychology
        this.timeframes = {
            short: {period: 7, weight: 0.5},    // 50% weight - most important
            medium: {period: 25, weight: 0.3},   // 30% weight
            long: {period: 99, weight: 0.2}      // 20% weight
        };
    }

    calculateScore() {
        // Let's use simple numbers for our example
        const currentPrice = 28;
        
        // Example moving averages:
        const averages = {
            short: 23.29,  // 7-day average
            medium: 21.50, // 25-day average
            long: 20.00    // 99-day average
        };

        // Calculate position scores
        const positions = {
            short: (28 - 23.29) / 23.29,   // = 0.202
            medium: (28 - 21.50) / 21.50,  // = 0.302
            long: (28 - 20.00) / 20.00     // = 0.400
        };

        // Example previous day averages
        const yesterdayAverages = {
            short: 22.50,
            medium: 21.00,
            long: 19.80
        };

        // Calculate trend strengths
        const trends = {
            short: (23.29 - 22.50) / 22.50,   // = 0.035
            medium: (21.50 - 21.00) / 21.00,  // = 0.024
            long: (20.00 - 19.80) / 19.80     // = 0.010
        };

        // Weighted position score
        const positionScore = 
            (positions.short * 0.5) +   // 0.202 * 0.5 = 0.101
            (positions.medium * 0.3) +  // 0.302 * 0.3 = 0.091
            (positions.long * 0.2);     // 0.400 * 0.2 = 0.080
        // Total = 0.272

        // Weighted trend score
        const trendScore = 
            (trends.short * 0.5) +   // 0.035 * 0.5 = 0.018
            (trends.medium * 0.3) +  // 0.024 * 0.3 = 0.007
            (trends.long * 0.2);     // 0.010 * 0.2 = 0.002
        // Total = 0.027

        // Final raw score
        const rawScore = (positionScore + trendScore) / 2;
        // = (0.272 + 0.027) / 2 = 0.150

        // Convert to -1 to 1 range using tanh
        return Math.tanh(rawScore);  // = 0.149
    }
}
```

### Interpreting the Final Score
In our example:
- Final Score = 0.149 (on -1 to 1 scale)
- This indicates mild greed/bullishness because:
  - Price is above all moving averages
  - All trends are positive
  - But the movement isn't extreme enough to suggest excessive greed

Real-world interpretation:
- Scores near 0 (like our 0.149) suggest cautious optimism
- Scores above 0.5 suggest strong greed
- Scores below -0.5 suggest strong fear
- Extreme scores (near 1 or -1) suggest potential market tops or bottoms