# Volatility Score Component Explained

## What is Volatility and Why it Matters
Volatility measures how wildly prices are swinging. High volatility often indicates fear in the market (panic selling or panic buying), while low volatility usually suggests calm and confidence. Think of it like measuring the market's anxiety level.

The Volatility Score combines two key measurements:
1. True Range (daily price swings)
2. Standard Deviation of Returns (how erratic price changes are over time)

## Breaking it Down with Simple Numbers

### 1. True Range (TR)
True Range captures the full trading range by considering:
- Today's high to low
- The gap from previous close to today's high
- The gap from previous close to today's low

#### Example with Real Numbers:
```javascript
// Example OHLC (Open, High, Low, Close) data:
const yesterday = {
    open: 100,
    high: 105,
    low: 98,
    close: 102
};

const today = {
    open: 103,
    high: 108,
    low: 101,
    close: 106
};

class VolatilityCalculator {
    calculateTrueRange(today, yesterday) {
        // Calculate three differences:
        const highLow = today.high - today.low;                    // 108 - 101 = 7
        const highClose = Math.abs(today.high - yesterday.close);  // |108 - 102| = 6
        const lowClose = Math.abs(today.low - yesterday.close);    // |101 - 102| = 1

        // True Range is the largest of these
        return Math.max(highLow, highClose, lowClose);  // TR = 7
    }
}
```

Real-world meaning:
- TR = 7 means the maximum price movement was $7
- Larger TR values indicate more volatile trading
- A series of increasing TRs suggests building market anxiety

### 2. Average True Range (ATR)
ATR smooths out True Range values over a period (usually 14 days) to show sustained volatility levels.

```javascript
// Example 5 days of True Range values:
const trueRanges = [7, 5, 8, 6, 9];

class VolatilityCalculator {
    calculateATR(trueRanges, period = 5) {
        // First ATR is simple average
        let atr = trueRanges.reduce((sum, tr) => sum + tr, 0) / period;
        // ATR = (7 + 5 + 8 + 6 + 9) / 5 = 7

        // Subsequent ATR uses smoothing:
        // ATR = (Previous ATR * (period-1) + Current TR) / period
        
        // If next day's TR is 10:
        // New ATR = (7 * 4 + 10) / 5 = 7.6
        return atr;
    }
}
```

### 3. Standard Deviation of Returns
This measures how much daily price changes vary from their average change.

```javascript
// Example 5 days of closing prices:
const prices = [100, 102, 99, 103, 101];

class VolatilityCalculator {
    calculateReturns(prices) {
        const returns = [];
        for (let i = 1; i < prices.length; i++) {
            // Calculate percentage change
            const return = Math.log(prices[i] / prices[i-1]);
            returns.push(return);
        }
        // Returns: [0.0198, -0.0303, 0.0398, -0.0198]
        return returns;
    }

    calculateStdDev(returns) {
        // 1. Calculate average return
        const mean = returns.reduce((sum, ret) => sum + ret, 0) / returns.length;
        // mean = 0.00238

        // 2. Calculate squared differences from mean
        const squaredDiffs = returns.map(ret => Math.pow(ret - mean, 2));

        // 3. Calculate variance (average of squared differences)
        const variance = squaredDiffs.reduce((sum, diff) => sum + diff, 0) / returns.length;

        // 4. Take square root to get standard deviation
        return Math.sqrt(variance);
        // In this example ≈ 0.0316 or 3.16%
    }
}
```

### Complete Implementation with Real Example

```javascript
class VolatilityCalculator {
    constructor(ohlcData, period = 14) {
        this.ohlcData = ohlcData;
        this.period = period;
    }

    calculate() {
        // Let's say we have calculated:
        const atr = 7.6;  // From our earlier example
        const maxATR = 10; // Highest ATR in last 14 days
        const stdDev = 0.0316; // From our returns example
        
        // 1. Normalize ATR to 0-1 scale
        const atrScore = atr / maxATR;  
        // = 7.6 / 10 = 0.76

        // 2. Normalize standard deviation
        // We multiply by sqrt(252) to annualize (252 trading days)
        const annualizedStdDev = stdDev * Math.sqrt(252);
        // = 0.0316 * sqrt(252) = 0.502 or 50.2% annual volatility
        
        // Cap at 200% annual volatility
        const stdDevScore = Math.min(annualizedStdDev / 2, 1);
        // = 0.502 / 2 = 0.251

        // 3. Combine scores (60% ATR, 40% StdDev)
        const rawVolatility = (atrScore * 0.6) + (stdDevScore * 0.4);
        // = (0.76 * 0.6) + (0.251 * 0.4)
        // = 0.456 + 0.100
        // = 0.556

        // 4. Apply exponential scaling for better sensitivity
        const finalScore = 1 - Math.exp(-3 * rawVolatility);
        // = 1 - e^(-3 * 0.556)
        // = 1 - 0.187
        // = 0.813

        return finalScore;
    }
}
```

### Interpreting the Volatility Score

In our example:
- Final Score = 0.813 (on 0 to 1 scale)
- This indicates high volatility because:
  - ATR is 76% of its recent maximum
  - Annual volatility is around 50%
  - The exponential scaling emphasizes this high volatility

Real-world interpretation:
- 0.0 - 0.3: Low volatility (market is calm)
- 0.3 - 0.6: Normal volatility (typical trading conditions)
- 0.6 - 0.8: High volatility (market is nervous)
- 0.8 - 1.0: Extreme volatility (market is panicking)

When combined with Price Score:
- High volatility + Positive price score = Greedy panic buying
- High volatility + Negative price score = Fearful panic selling
- Low volatility + Positive price score = Confident uptrend
- Low volatility + Negative price score = Resigned downtrend