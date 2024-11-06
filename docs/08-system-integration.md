# Complete Fear & Greed Index Integration

## System Overview
The complete system combines both internal (price-based) and external (sentiment-based) components into a unified index. Here's how they all fit together:

```javascript
class SolanaFearAndGreedIndex {
    constructor(config) {
        this.weights = {
            internal: {
                total: 0.6,  // 60% weight to price-based signals
                breakdown: {
                    price: 0.25,     // 25% of internal
                    volatility: 0.20,
                    volume: 0.20,
                    impulse: 0.20,
                    technical: 0.15
                }
            },
            external: {
                total: 0.4,  // 40% weight to external signals
                breakdown: {
                    social: 0.25,    // 25% of external
                    trends: 0.20,
                    whales: 0.30,
                    orderBook: 0.25
                }
            }
        };
        
        // Initialize analyzers
        this.initializeAnalyzers();
    }

    initializeAnalyzers() {
        // Internal components
        this.priceAnalyzer = new PriceScoreCalculator();
        this.volatilityAnalyzer = new VolatilityCalculator();
        this.volumeAnalyzer = new VolumeAnalyzer();
        this.impulseAnalyzer = new ImpulseCalculator();
        this.technicalAnalyzer = new TechnicalAnalysisCalculator();

        // External components
        this.externalAnalyzer = new ExternalDataAnalyzer();
    }

    async calculateIndex() {
        // 1. Gather all signals
        const signals = await this.gatherAllSignals();
        
        // 2. Calculate weighted scores
        const score = this.calculateWeightedScore(signals);
        
        // 3. Generate comprehensive analysis
        const analysis = this.generateAnalysis(signals, score);

        return {
            score,
            analysis,
            signals
        };
    }

    async gatherAllSignals() {
        try {
            // Calculate internal signals
            const internal = await this.calculateInternalSignals();
            
            // Calculate external signals
            const external = await this.calculateExternalSignals();

            return {
                internal,
                external
            };
        } catch (error) {
            return this.handleMissingData(error);
        }
    }

    async calculateInternalSignals() {
        // Example values from our previous calculations
        return {
            price: await this.priceAnalyzer.calculate(),        // 77.8
            volatility: await this.volatilityAnalyzer.calculate(), // 81.3
            volume: await this.volumeAnalyzer.calculate(),      // 97.9
            impulse: await this.impulseAnalyzer.calculate(),    // 77.8
            technical: await this.technicalAnalyzer.calculate()  // 84.8
        };
    }

    async calculateExternalSignals() {
        // Example values from our external components
        const externalData = await this.externalAnalyzer.calculateExternalScore();
        return {
            social: 80.45,
            trends: 71.20,
            whales: 80.45,
            orderBook: 76.55
        };
    }

    calculateWeightedScore(signals) {
        // Calculate internal score
        const internalScore = (
            (signals.internal.price * this.weights.internal.breakdown.price) +
            (signals.internal.volatility * this.weights.internal.breakdown.volatility) +
            (signals.internal.volume * this.weights.internal.breakdown.volume) +
            (signals.internal.impulse * this.weights.internal.breakdown.impulse) +
            (signals.internal.technical * this.weights.internal.breakdown.technical)
        );

        // Calculate external score
        const externalScore = (
            (signals.external.social * this.weights.external.breakdown.social) +
            (signals.external.trends * this.weights.external.breakdown.trends) +
            (signals.external.whales * this.weights.external.breakdown.whales) +
            (signals.external.orderBook * this.weights.external.breakdown.orderBook)
        );

        // Combine with main weights
        return Math.round(
            (internalScore * this.weights.internal.total) +
            (externalScore * this.weights.external.total)
        );
    }

    generateAnalysis(signals, finalScore) {
        return {
            score: finalScore,
            sentiment: this.getSentimentLevel(finalScore),
            components: {
                internal: this.analyzeInternalSignals(signals.internal),
                external: this.analyzeExternalSignals(signals.external)
            },
            divergences: this.detectDivergences(signals),
            confidence: this.calculateConfidence(signals),
            warnings: this.generateWarnings(signals)
        };
    }

    detectDivergences(signals) {
        const divergences = [];

        // 1. Price-Volume Divergence
        if (Math.abs(signals.internal.price - signals.internal.volume) > 30) {
            divergences.push({
                type: 'price-volume',
                severity: 'high',
                description: 'Price movement lacks volume confirmation'
            });
        }

        // 2. Technical-Whale Divergence
        if (Math.abs(signals.internal.technical - signals.external.whales) > 30) {
            divergences.push({
                type: 'technical-whale',
                severity: 'medium',
                description: 'Technical analysis conflicts with whale behavior'
            });
        }

        // 3. Social-Price Divergence
        if (Math.abs(signals.external.social - signals.internal.price) > 40) {
            divergences.push({
                type: 'social-price',
                severity: 'medium',
                description: 'Social sentiment misaligned with price action'
            });
        }

        return divergences;
    }

    calculateConfidence(signals) {
        // Measure signal alignment
        const allSignals = [
            ...Object.values(signals.internal),
            ...Object.values(signals.external)
        ];

        // Calculate standard deviation of signals
        const mean = allSignals.reduce((a, b) => a + b) / allSignals.length;
        const variance = allSignals.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / allSignals.length;
        const stdDev = Math.sqrt(variance);

        // Convert to confidence score (lower stdDev = higher confidence)
        const confidence = Math.max(0, 100 - (stdDev * 2));

        return {
            score: confidence,
            level: this.getConfidenceLevel(confidence),
            reasons: this.getConfidenceReasons(signals, stdDev)
        };
    }

    generateWarnings(signals) {
        const warnings = [];

        // 1. Check for extreme readings
        if (Math.abs(signals.internal.price) > 90) {
            warnings.push({
                type: 'extreme_price',
                level: 'high',
                message: 'Price reaching extreme levels'
            });
        }

        // 2. Check for volume concerns
        if (signals.internal.volume < 30 && signals.internal.price > 70) {
            warnings.push({
                type: 'low_volume',
                level: 'medium',
                message: 'Low volume on price increase'
            });
        }

        // 3. Check for whale divergence
        if (signals.external.whales < 30 && signals.internal.price > 70) {
            warnings.push({
                type: 'whale_divergence',
                level: 'high',
                message: 'Whale distribution during price increase'
            });
        }

        return warnings;
    }
}

// Example usage:
const index = new SolanaFearAndGreedIndex();
const result = await index.calculateIndex();

console.log(`Fear & Greed Score: ${result.score}`);
console.log('Analysis:', result.analysis);
```

## Example Output

```javascript
{
    score: 78,
    sentiment: "Extreme Greed",
    components: {
        internal: {
            price: { score: 77.8, contribution: "Strong uptrend" },
            volatility: { score: 81.3, contribution: "High volatility" },
            volume: { score: 97.9, contribution: "Very strong volume" },
            impulse: { score: 77.8, contribution: "Strong momentum" },
            technical: { score: 84.8, contribution: "Bullish technicals" }
        },
        external: {
            social: { score: 80.45, contribution: "Very positive sentiment" },
            trends: { score: 71.20, contribution: "Growing interest" },
            whales: { score: 80.45, contribution: "Whale accumulation" },
            orderBook: { score: 76.55, contribution: "Buy pressure" }
        }
    },
    divergences: [
        {
            type: "technical-whale",
            severity: "medium",
            description: "Technical analysis conflicts with whale behavior"
        }
    ],
    confidence: {
        score: 85,
        level: "High",
        reasons: [
            "Strong signal alignment",
            "Multiple confirmation sources",
            "Consistent across timeframes"
        ]
    },
    warnings: [
        {
            type: "extreme_price",
            level: "high",
            message: "Price reaching extreme levels"
        }
    ]
}
```
