# Crypto Fear and Greed Index

A comprehensive implementation of a Crypto Fear and Greed Index with multiple data sources, technical analysis, and trading strategies. This project provides tools to analyze market sentiment through price action, social signals, and on-chain metrics.

## Features

- Multi-factor sentiment analysis combining:
  - Price action metrics
  - Technical indicators
  - Social sentiment
  - Search trends
  - Whale wallet tracking
  - Order book analysis
- Trading strategy generation
- Position sizing recommendations
- Real-time updates

## Documentation

### [Price Score](docs/01-price-score.md)
Complete analysis of price trends across multiple timeframes. This component measures market direction and strength using moving averages and trend analysis, providing a foundation for sentiment scoring.
Data Source can be: Jupiter Price API, run signal on historical data.

### [Volatility Score](docs/02-volatility-score.md)
Measures market volatility using True Range and standard deviation of returns. This component helps gauge market risk and sentiment extremes through price movement patterns.
Data Source can be: Jupiter Price API, run signal on historical data.

### [Volume Analysis](docs/03-volume-analysis.md)
Deep dive into trading volume patterns and their relationship with price movement. This component analyzes volume trends, breakouts, and divergences to confirm market sentiment.
Data Source can be: Jupiter Price API, run signal on historical data.

### [Impulse Score](docs/04-impulse-score.md)
Measures the momentum and force of price movements using RSI, MACD, and Rate of Change indicators. This component helps identify the strength and sustainability of trends.
Data Source can be: Jupiter Price API, run signal on historical data.

### [Technical Analysis](docs/05-technical-analysis.md)
Comprehensive technical analysis incorporating multiple indicators and oscillators. This component provides a broad market view using traditional technical analysis tools.
Data Source can be: Jupiter Price API, run signal on historical data.

### [Social Sentiment](docs/06-social-sentiment.md)
Analysis of social media sentiment across multiple platforms. This component tracks and measures market sentiment through social signals, engagement metrics, and linguistic analysis.
Data Source can be: Possible AI Interpretation of Twitter, Reddit, and LunarCrush Social Metrics.

### [External Components](docs/07-external-components.md)
Integration of external data sources including Google Trends, whale wallet tracking, and order book analysis. These components provide additional context and confirmation for market sentiment.

### [System Integration](docs/08-system-integration.md)
Detailed explanation of how all components are weighted and combined into the final index. This document covers the complete system architecture and component interaction.

### [Trading Strategies](docs/09-trading-strategies.md)
Implementation of trading strategies based on the Fear and Greed Index. This component provides position sizing, entry/exit rules, and risk management guidelines.

### [Social Sentiment Using AI](docs/10-social-sentiment-using-ai.md)
Alternative implementation of social sentiment analysis using Anthropic's Claude 3.5 Haiku API. This component provides deep learning-based sentiment analysis of social media content across Reddit, Twitter, and Telegram, with detailed metrics and confidence scoring. Perfect for teams that prefer AI-based sentiment analysis over traditional methods.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contact

Project Link: [https://github.com/SurfSolana/Crypto-Fear-And-Greed-Index](https://github.com/SurfSolana/Crypto-Fear-And-Greed-Index)

## Disclaimer

This tool is for informational purposes only. Never make investment decisions purely based on the Fear and Greed Index. Always do your own research and understand the risks involved in cryptocurrency trading.
