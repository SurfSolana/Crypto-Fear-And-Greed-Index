# Impulse Score Component Explained

## What is Impulse and Why it Matters
Impulse measures the momentum and force of price movements. While the Price Score tells us where we are, Impulse tells us how we got here. It's like measuring not just a car's position, but its speed and acceleration.

This matters because:
- Strong impulse in either direction often precedes continued movement
- Weakening impulse can signal potential reversals
- Multiple impulse signals together provide confirmation of market sentiment

## Breaking it Down with Components

### 1. RSI (Relative Strength Index)
RSI measures whether Solana is overbought or oversold by comparing the magnitude of recent gains to recent losses.

```javascript
class ImpulseCalculator {
    /**
     * Calculate RSI using multiple timeframes
     */
    calculateRSI(prices, period) {
        // Let's use real numbers for 14 days of price changes:
        const priceChanges = [
            2,  // Up
            1,  // Up
            -3, // Down
            4,  // Up
            -1, // Down
            2,  // Up
            3,  // Up
            -2, // Down
            1,  // Up
            -1, // Down
            3,  // Up
            2,  // Up
            -2, // Down
            1   // Up
        ];

        // 1. Calculate average gains and losses
        let avgGain = 0;
        let avgLoss = 0;

        // First, get initial averages
        priceChanges.slice(0, period).forEach(change => {
            if (change > 0) avgGain += change;
            if (change < 0) avgLoss += Math.abs(change);
        });

        avgGain = avgGain / period;  // = (2 + 1 + 4 + 2 + 3 + 1 + 3 + 2 + 1) / 14 = 1.36
        avgLoss = avgLoss / period;  // = (3 + 1 + 2 + 1 + 2) / 14 = 0.64

        // 2. Calculate RS (Relative Strength)
        const RS = avgGain / avgLoss;  // = 1.36 / 0.64 = 2.125

        // 3. Calculate RSI
        const RSI = 100 - (100 / (1 + RS));
        // = 100 - (100 / 3.125)
        // = 100 - 32
        // = 68

        return RSI;  // 68 indicates slightly overbought conditions
    }

    /**
     * Calculate RSI for multiple timeframes and combine them
     */
    calculateMultipleRSI(prices) {
        const shortRSI = this.calculateRSI(prices, 14);    // = 68
        const mediumRSI = this.calculateRSI(prices, 25);   // = 65 (example)
        const longRSI = this.calculateRSI(prices, 50);     // = 60 (example)

        // Convert RSI values to -1 to 1 scale
        const rsiScores = {
            short: (shortRSI - 50) / 50,   // = (68 - 50) / 50 = 0.36
            medium: (mediumRSI - 50) / 50, // = (65 - 50) / 50 = 0.30
            long: (longRSI - 50) / 50      // = (60 - 50) / 50 = 0.20
        };

        // Weight the different timeframes
        const weightedRSI = 
            (rsiScores.short * 0.5) +   // 0.36 * 0.5 = 0.180
            (rsiScores.medium * 0.3) +  // 0.30 * 0.3 = 0.090
            (rsiScores.long * 0.2);     // 0.20 * 0.2 = 0.040
        
        return weightedRSI;  // = 0.310 (moderately positive momentum)
    }
}
```

### 2. MACD (Moving Average Convergence Divergence)
MACD measures the relationship between two moving averages of prices, showing momentum and potential trend changes.

```javascript
class ImpulseCalculator {
    calculateMACD(prices) {
        // Example closing prices for 26 days
        const shortPeriod = 12;
        const longPeriod = 26;
        const signalPeriod = 9;

        // Let's say we've calculated:
        const ema12 = 105;  // 12-day EMA
        const ema26 = 102;  // 26-day EMA

        // 1. Calculate MACD Line
        const macdLine = ema12 - ema26;  // = 105 - 102 = 3

        // 2. Calculate Signal Line (9-day EMA of MACD Line)
        const signalLine = 2;  // Example value

        // 3. Calculate MACD Histogram
        const macdHist = macdLine - signalLine;  // = 3 - 2 = 1

        // 4. Normalize histogram to -1 to 1 scale
        // Using recent maximum histogram value for normalization
        const maxHist = 5;  // Example maximum from recent history
        const macdScore = macdHist / maxHist;  // = 1 / 5 = 0.2

        return macdScore;  // 0.2 indicates moderate positive momentum
    }
}
```

### 3. Price Rate of Change (ROC)
ROC measures the speed of price changes, which helps identify acceleration or deceleration in trends.

```javascript
class ImpulseCalculator {
    calculateROC(prices) {
        // Example: Calculate 7-day and 25-day ROC
        // Let's say current price is 100
        const currentPrice = 100;
        
        // Prices from 7 and 25 days ago
        const price7daysAgo = 90;
        const price25daysAgo = 80;

        // Calculate ROC percentages
        const shortROC = (currentPrice - price7daysAgo) / price7daysAgo;
        // = (100 - 90) / 90 = 0.111 or 11.1%

        const mediumROC = (currentPrice - price25daysAgo) / price25daysAgo;
        // = (100 - 80) / 80 = 0.250 or 25%

        // Weight the ROC values
        const weightedROC = 
            (shortROC * 0.7) +    // 0.111 * 0.7 = 0.078
            (mediumROC * 0.3);    // 0.250 * 0.3 = 0.075
        
        // = 0.153

        // Normalize using tanh for -1 to 1 scale
        return Math.tanh(weightedROC * 3);  // = tanh(0.459) = 0.430
    }
}
```

### Complete Implementation

```javascript
class ImpulseCalculator {
    constructor(prices) {
        this.prices = prices;
    }

    calculate() {
        // Using our previous example calculations:
        const rsiScore = 0.310;    // From RSI calculation
        const macdScore = 0.200;   // From MACD calculation
        const rocScore = 0.430;    // From ROC calculation

        // Combine all components with weights
        const impulseScore = (
            (rsiScore * 0.4) +     // 0.310 * 0.4 = 0.124
            (macdScore * 0.3) +    // 0.200 * 0.3 = 0.060
            (rocScore * 0.3)       // 0.430 * 0.3 = 0.129
        );
        // = 0.313

        // Final normalization to ensure -1 to 1 range
        return Math.tanh(impulseScore * 2);  // = tanh(0.626) = 0.556
    }
}
```

### Interpreting the Impulse Score

In our example:
- Final Score = 0.556 (on -1 to 1 scale)
- This indicates moderate positive momentum because:
  - RSI shows bullish momentum (0.310)
  - MACD confirms trend (0.200)
  - ROC shows accelerating prices (0.430)

Real-world interpretation:
- -1.0 to -0.7: Strong negative momentum (Fear accelerating)
- -0.7 to -0.3: Moderate negative momentum (Fear building)
- -0.3 to 0.3: Neutral momentum (Sentiment stabilizing)
- 0.3 to 0.7: Moderate positive momentum (Greed building)
- 0.7 to 1.0: Strong positive momentum (Greed accelerating)

Key Insights:
1. Momentum Confirmation
   - When Price Score and Impulse Score align, trend is strong
   - When they diverge, potential reversal incoming

2. Extreme Readings
   - Scores near ±1.0 often signal unsustainable momentum
   - Look for momentum divergences at price extremes

3. Cross-Component Analysis
   - High Impulse + High Volume = Strong trend confirmation
   - High Impulse + Low Volume = Potential fake out
   - Low Impulse + High Volume = Potential trend change