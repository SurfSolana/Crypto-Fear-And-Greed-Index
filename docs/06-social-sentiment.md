# Social Media Sentiment Analysis Component

## What is Social Sentiment and Why it Matters
Social sentiment measures the mood and opinion of Solana traders and holders across social media platforms. It's important because:
- Social sentiment often precedes price movements
- Strong shifts in sentiment can predict trend changes
- Volume of social activity indicates market interest

## Breaking Down Social Sentiment

### 1. Platform-Specific Sentiment Scoring

```javascript
class SocialSentimentAnalyzer {
    constructor(socialData) {
        this.weights = {
            twitter: 0.40,    // 40% weight - most real-time
            reddit: 0.35,     // 35% weight - detailed discussion
            telegram: 0.25    // 25% weight - trading focused
        };
    }

    analyzePlatform(posts, platform) {
        // Example data structure for a post:
        const examplePost = {
            text: "Solana's network performance is amazing! Bullish! 🚀",
            likes: 150,
            replies: 45,
            reposts: 28,
            timestamp: "2024-03-15T10:30:00Z"
        };

        // 1. Calculate engagement score
        const calculateEngagement = (post) => {
            const engagementScore = (
                post.likes * 1 +      // Base engagement
                post.replies * 2 +    // Replies weighted more
                post.reposts * 3      // Reposts weighted highest
            );
            
            // Normalize by average engagement levels
            const avgEngagement = 100; // Platform-specific average
            return engagementScore / avgEngagement;
        };

        // 2. Sentiment analysis with engagement weighting
        let weightedSentiment = 0;
        let totalWeight = 0;

        posts.forEach(post => {
            const engagement = calculateEngagement(post);
            const sentiment = this.analyzeSinglePost(post);
            
            weightedSentiment += sentiment * engagement;
            totalWeight += engagement;
        });

        return weightedSentiment / totalWeight;
    }

    analyzeSinglePost(post) {
        // Example sentiment analysis implementation
        
        // 1. Pre-process text
        const processedText = this.preprocessText(post.text);
        
        // 2. Score different aspects
        const scores = {
            // Keywords scoring
            keywords: this.scoreKeywords(processedText),
            
            // Emoji sentiment
            emojis: this.scoreEmojis(post.text),
            
            // Technical terms sentiment
            technical: this.scoreTechnicalTerms(processedText),
            
            // Price discussion sentiment
            price: this.scorePriceDiscussion(processedText)
        };

        // Example calculation:
        // Let's say we have these scores:
        const exampleScores = {
            keywords: 0.8,    // Very positive keywords
            emojis: 0.7,     // Bullish emojis
            technical: 0.5,   // Neutral technical discussion
            price: 0.6       // Slightly bullish price discussion
        };

        // Weight the different aspects
        return (
            exampleScores.keywords * 0.35 +
            exampleScores.emojis * 0.15 +
            exampleScores.technical * 0.25 +
            exampleScores.price * 0.25
        );
        // = 0.8*0.35 + 0.7*0.15 + 0.5*0.25 + 0.6*0.25
        // = 0.28 + 0.105 + 0.125 + 0.15
        // = 0.66 (positive sentiment)
    }

    scoreKeywords(text) {
        const bullishTerms = new Set([
            'bullish', 'moon', 'buying', 'accumulate', 'undervalued',
            'strong', 'higher', 'support', 'breakout', 'momentum'
        ]);

        const bearishTerms = new Set([
            'bearish', 'dump', 'selling', 'exit', 'overvalued',
            'weak', 'lower', 'resistance', 'breakdown', 'fail'
        ]);

        let bullishCount = 0;
        let bearishCount = 0;

        // Count term occurrences
        text.split(' ').forEach(word => {
            if (bullishTerms.has(word.toLowerCase())) bullishCount++;
            if (bearishTerms.has(word.toLowerCase())) bearishCount++;
        });

        // Calculate sentiment score (-1 to 1)
        const total = bullishCount + bearishCount;
        if (total === 0) return 0;

        return (bullishCount - bearishCount) / total;
    }

    calculateOverallSentiment(platformData) {
        // Example data:
        const sentiments = {
            twitter: 0.66,    // From our previous calculation
            reddit: 0.45,     // More measured sentiment
            telegram: 0.75    // More extreme sentiment
        };

        // Weight and combine platform sentiments
        const weightedSentiment = 
            (sentiments.twitter * this.weights.twitter) +
            (sentiments.reddit * this.weights.reddit) +
            (sentiments.telegram * this.weights.telegram);
        
        // = 0.66*0.40 + 0.45*0.35 + 0.75*0.25
        // = 0.264 + 0.1575 + 0.1875
        // = 0.609

        // Convert to 0-100 scale
        return (weightedSentiment + 1) * 50;  // = 80.45
    }

    generateSocialMetrics() {
        return {
            overallSentiment: this.calculateOverallSentiment(),
            volumeMetrics: this.calculateSocialVolume(),
            topicAnalysis: this.analyzeTopics(),
            influencerSentiment: this.analyzeInfluencers()
        };
    }
}
```

### Social Sentiment Score Interpretation

1. Raw Score Meanings (0-100):
   - 0-20: Extremely Negative
   - 21-40: Negative
   - 41-60: Neutral
   - 61-80: Positive
   - 81-100: Extremely Positive

2. Volume Considerations:
   - High volume + Strong sentiment = Stronger signal
   - Low volume + Strong sentiment = Weaker signal
   - Sudden volume spikes = Potential trend change

3. Topic Analysis:
   - Network performance discussions
   - Development activity
   - Competitor comparisons
   - Price predictions
   - Trading discussions

4. Platform-Specific Insights:
   - Twitter: Quick sentiment shifts
   - Reddit: Deeper analysis and discussion
   - Telegram: Trading sentiment