# Social Sentiment Analysis Using Claude 3.5 Haiku

## Overview
This implementation uses the Anthropic Claude 3.5 Haiku API to analyze sentiment from multiple social media sources. We use Claude's natural language processing capabilities to analyze posts, comments, and discussions about cryptocurrencies across platforms.

## Implementation

```javascript
import Anthropic from '@anthropic-ai/sdk';

class ClaudeSentimentAnalyzer {
    constructor(config) {
        this.anthropic = new Anthropic({
            apiKey: config.anthropicApiKey
        });
        
        this.systemPrompt = `
            You are a cryptocurrency sentiment analyst. Analyze the provided social media content
            and rate the sentiment on a scale from -1 (extremely bearish) to 1 (extremely bullish).
            Consider:
            - Technical analysis mentions
            - Price predictions
            - Project fundamentals
            - Market psychology
            - Emotional tone
            Return a JSON object with the sentiment score and detailed analysis.
        `;
    }

    /**
     * Process batch of social media posts
     */
    async analyzeBatch(posts) {
        const batchResults = [];
        
        // Process in chunks to avoid rate limits
        const chunks = this.chunkArray(posts, 10);
        
        for (const chunk of chunks) {
            const chunkResults = await Promise.all(
                chunk.map(post => this.analyzePost(post))
            );
            batchResults.push(...chunkResults);
        }
        
        return this.aggregateResults(batchResults);
    }

    /**
     * Analyze single social media post
     */
    async analyzePost(post) {
        try {
            const response = await this.anthropic.messages.create({
                model: 'claude-3-haiku-20240307',
                max_tokens: 1024,
                temperature: 0.2,
                system: this.systemPrompt,
                messages: [{
                    role: 'user',
                    content: this.formatPostForAnalysis(post)
                }]
            });

            return JSON.parse(response.content[0].text);

        } catch (error) {
            console.error('Error analyzing post:', error);
            return {
                sentiment: 0,
                confidence: 0,
                error: error.message
            };
        }
    }

    /**
     * Format post data for Claude analysis
     */
    formatPostForAnalysis(post) {
        return `
            Analyze the sentiment of this cryptocurrency-related social media post:
            
            Platform: ${post.platform}
            Content: ${post.content}
            Likes/Votes: ${post.engagement.likes}
            Comments/Replies: ${post.engagement.comments}
            Shares/Retweets: ${post.engagement.shares}
            Timestamp: ${post.timestamp}
            
            Additional context:
            - User influence score: ${post.userInfluence}
            - Contains links: ${post.hasLinks}
            - Contains media: ${post.hasMedia}
            
            Provide a detailed sentiment analysis in JSON format including:
            - Overall sentiment score (-1 to 1)
            - Confidence score (0 to 1)
            - Key sentiment drivers
            - Technical analysis mentions
            - Notable price predictions
            - Emotional tone assessment
        `;
    }

    /**
     * Process Reddit-specific content
     */
    async analyzeRedditContent(subredditData) {
        const posts = subredditData.map(item => ({
            platform: 'reddit',
            content: `${item.title}\n${item.selftext}`,
            engagement: {
                likes: item.score,
                comments: item.num_comments,
                shares: 0
            },
            userInfluence: this.calculateRedditUserInfluence(item.author),
            hasLinks: item.url !== item.permalink,
            hasMedia: item.is_video || item.media !== null,
            timestamp: item.created_utc
        }));

        return this.analyzeBatch(posts);
    }

    /**
     * Process Twitter/X-specific content
     */
    async analyzeTwitterContent(tweets) {
        const posts = tweets.map(tweet => ({
            platform: 'twitter',
            content: tweet.text,
            engagement: {
                likes: tweet.favorite_count,
                comments: tweet.reply_count,
                shares: tweet.retweet_count
            },
            userInfluence: this.calculateTwitterUserInfluence(tweet.user),
            hasLinks: tweet.entities.urls.length > 0,
            hasMedia: tweet.entities.media?.length > 0,
            timestamp: tweet.created_at
        }));

        return this.analyzeBatch(posts);
    }

    /**
     * Process Telegram-specific content
     */
    async analyzeTelegramContent(messages) {
        const posts = messages.map(msg => ({
            platform: 'telegram',
            content: msg.text,
            engagement: {
                likes: msg.reactions?.total_count || 0,
                comments: msg.reply_count || 0,
                shares: msg.forwards || 0
            },
            userInfluence: this.calculateTelegramUserInfluence(msg.from),
            hasLinks: msg.entities?.some(e => e.type === 'url'),
            hasMedia: msg.photo || msg.video,
            timestamp: msg.date
        }));

        return this.analyzeBatch(posts);
    }

    /**
     * Aggregate results from multiple posts
     */
    aggregateResults(results) {
        const validResults = results.filter(r => !r.error);
        
        if (validResults.length === 0) return {
            sentiment: 0,
            confidence: 0,
            error: 'No valid results to analyze'
        };

        // Weight results by confidence and engagement
        const weightedSentiments = validResults.map(result => ({
            sentiment: result.sentiment,
            weight: result.confidence * this.calculateEngagementWeight(result)
        }));

        const totalWeight = weightedSentiments.reduce((sum, item) => sum + item.weight, 0);
        const weightedSentiment = weightedSentiments.reduce((sum, item) => 
            sum + (item.sentiment * item.weight), 0) / totalWeight;

        return {
            sentiment: weightedSentiment,
            confidence: this.calculateAggregateConfidence(validResults),
            details: this.generateSentimentDetails(validResults),
            metrics: this.calculateSentimentMetrics(validResults)
        };
    }

    /**
     * Generate detailed sentiment analysis report
     */
    generateSentimentDetails(results) {
        const technicalAnalysis = results
            .filter(r => r.technicalAnalysisMentions)
            .map(r => r.technicalAnalysisMentions);

        const pricePredictions = results
            .filter(r => r.pricePredictions)
            .map(r => r.pricePredictions);

        const emotionalTones = results
            .filter(r => r.emotionalTone)
            .map(r => r.emotionalTone);

        return {
            technicalAnalysis: this.summarizeTechnicalAnalysis(technicalAnalysis),
            pricePredictions: this.summarizePricePredictions(pricePredictions),
            emotionalTone: this.summarizeEmotionalTones(emotionalTones),
            topKeywords: this.extractTopKeywords(results),
            sentimentDrivers: this.extractSentimentDrivers(results)
        };
    }

    /**
     * Calculate metrics for sentiment analysis
     */
    calculateSentimentMetrics(results) {
        return {
            totalPosts: results.length,
            averageConfidence: results.reduce((sum, r) => sum + r.confidence, 0) / results.length,
            sentimentDistribution: {
                bullish: results.filter(r => r.sentiment > 0.3).length,
                neutral: results.filter(r => r.sentiment >= -0.3 && r.sentiment <= 0.3).length,
                bearish: results.filter(r => r.sentiment < -0.3).length
            },
            platformBreakdown: this.calculatePlatformMetrics(results)
        };
    }
}

// Example usage:
const analyzer = new ClaudeSentimentAnalyzer({
    anthropicApiKey: 'your-api-key'
});

// Process content from multiple platforms
async function analyzeSocialSentiment() {
    // Fetch data from social platforms
    const redditData = await fetchRedditData();
    const twitterData = await fetchTwitterData();
    const telegramData = await fetchTelegramData();

    // Analyze each platform
    const [redditResults, twitterResults, telegramResults] = await Promise.all([
        analyzer.analyzeRedditContent(redditData),
        analyzer.analyzeTwitterContent(twitterData),
        analyzer.analyzeTelegramContent(telegramData)
    ]);

    // Combine results with platform weights
    const combinedResults = {
        reddit: {
            weight: 0.35,
            data: redditResults
        },
        twitter: {
            weight: 0.40,
            data: twitterResults
        },
        telegram: {
            weight: 0.25,
            data: telegramResults
        }
    };

    return analyzer.combineMultiPlatformResults(combinedResults);
}
```

## Example Output

```javascript
{
    "sentiment": 0.423,
    "confidence": 0.85,
    "details": {
        "technicalAnalysis": {
            "bullishPatterns": ["Golden Cross", "Cup and Handle"],
            "bearishPatterns": ["Rising Wedge"],
            "mentionedIndicators": ["RSI", "MACD", "EMA"]
        },
        "pricePredictions": {
            "shortTerm": {
                "bullish": 65,
                "bearish": 35,
                "averageTarget": 35000
            },
            "longTerm": {
                "bullish": 80,
                "bearish": 20,
                "averageTarget": 50000
            }
        },
        "emotionalTone": {
            "primary": "optimistic",
            "secondary": "cautious",
            "intensity": 0.7
        },
        "topKeywords": [
            "bullish",
            "accumulation",
            "support",
            "breakout",
            "adoption"
        ],
        "sentimentDrivers": [
            "Technical breakout potential",
            "Institutional adoption news",
            "Market cycle analysis",
            "Volume profile"
        ]
    },
    "metrics": {
        "totalPosts": 1250,
        "averageConfidence": 0.85,
        "sentimentDistribution": {
            "bullish": 650,
            "neutral": 450,
            "bearish": 150
        },
        "platformBreakdown": {
            "reddit": {
                "sentiment": 0.38,
                "confidence": 0.82,
                "volume": 450
            },
            "twitter": {
                "sentiment": 0.45,
                "confidence": 0.88,
                "volume": 600
            },
            "telegram": {
                "sentiment": 0.44,
                "confidence": 0.85,
                "volume": 200
            }
        }
    }
}
```

## Key Features

1. Multi-platform Analysis
   - Reddit, Twitter/X, and Telegram support
   - Platform-specific engagement metrics
   - Weighted influence scoring

2. Advanced Sentiment Analysis
   - Technical analysis detection
   - Price prediction extraction
   - Emotional tone analysis
   - Confidence scoring

3. Engagement Weighting
   - User influence consideration
   - Platform-specific metrics
   - Content type weighting

4. Error Handling
   - Rate limiting management
   - Batch processing
   - Error recovery

5. Detailed Analytics
   - Sentiment distribution
   - Platform breakdown
   - Confidence metrics
   - Key drivers identification

## Integration with Main Index

This component can be integrated into the main Fear and Greed Index by using the sentiment score and confidence metrics:

```javascript
class FearAndGreedIndex {
    async calculateSocialComponent() {
        const socialAnalysis = await this.claudeAnalyzer.analyzeSocialSentiment();
        
        // Convert -1 to 1 sentiment to 0 to 100 scale
        const normalizedScore = (socialAnalysis.sentiment + 1) * 50;
        
        // Weight by confidence
        const weightedScore = normalizedScore * socialAnalysis.confidence;
        
        return {
            score: weightedScore,
            confidence: socialAnalysis.confidence,
            details: socialAnalysis.details,
            metrics: socialAnalysis.metrics
        };
    }
}
```

## Notes

1. API Key Management:
   - Store your Anthropic API key securely
   - Monitor API usage and costs
   - Implement rate limiting

2. Performance Optimization:
   - Use batch processing for multiple posts
   - Cache frequent analyses
   - Implement retry logic

3. Error Handling:
   - Handle API timeouts
   - Manage rate limits
   - Process partial results

4. Regular Updates:
   - Update system prompt periodically
   - Fine-tune weights based on performance
   - Monitor accuracy metrics
