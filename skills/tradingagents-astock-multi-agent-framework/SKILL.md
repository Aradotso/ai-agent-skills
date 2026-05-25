---
name: tradingagents-astock-multi-agent-framework
description: A-share multi-agent investment research framework with 7 AI analysts, bull/bear debate, and risk assessment adapted for Chinese stock market
triggers:
  - analyze Chinese stock with AI agents
  - set up A-share trading analysis framework
  - run multi-agent stock research for A-shares
  - configure TradingAgents for Chinese market
  - analyze stock with 7 AI analysts
  - implement bull bear debate for stock analysis
  - use mootdx for A-share data
  - create trading decision pipeline with LLM agents
---

# TradingAgents-Astock Multi-Agent Framework

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

## Overview

TradingAgents-Astock is a multi-agent investment research framework specifically adapted for Chinese A-share markets. It orchestrates 7 specialized AI analyst agents that generate research reports, engage in bull/bear debates, perform risk assessment, and produce trading decisions. The framework handles A-share specific constraints (T+1 settlement, price limits, minimum lots) and uses free Chinese data sources (mootdx, EastMoney, Sina, THS) instead of Western APIs.

**Key Features:**
- 7 specialized analysts (Market, Social, News, Fundamentals, Policy, Hot Money, Lockup)
- Bull vs Bear research debate system
- 3-way risk assessment (Aggressive, Conservative, Neutral)
- A-share trading constraints (T+1, 涨跌停, minimum lots, ST rules)
- Dual LLM architecture (quick_think + deep_think)
- Web UI with real-time progress tracking
- Chinese output with English internal reasoning

## Installation

```bash
# Clone the repository
git clone https://github.com/simonlin1212/tradingagents-astock.git
cd tradingagents-astock

# Install base package (Python >= 3.10)
pip install -e .

# Optional: Install with Google Gemini support
pip install -e ".[google]"
```

## Configuration

### LLM Provider Setup

Create a `.env` file in the project root with your chosen LLM provider:

```bash
# MiniMax (Recommended for China, cost-effective)
MINIMAX_API_KEY=sk-your-key-here

# DeepSeek
DEEPSEEK_API_KEY=sk-your-key-here

# Zhipu GLM
ZHIPU_API_KEY=your-key-here

# Alibaba Qwen
DASHSCOPE_API_KEY=sk-your-key-here

# OpenAI
OPENAI_API_KEY=sk-your-key-here

# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-key-here

# Kimi (uses Anthropic-compatible API)
ANTHROPIC_AUTH_TOKEN=your-kimi-token
```

### Graph Configuration Object

```python
config = {
    "llm_provider": "minimax",              # Provider: minimax, deepseek, zhipu, qwen, openai, anthropic, google, xai, ollama
    "deep_think_llm": "MiniMax-M2.7",       # Model for Research Manager & Portfolio Manager
    "quick_think_llm": "MiniMax-M2.7-highspeed",  # Model for analysts, researchers, traders
    "output_language": "Chinese",            # Final report language (Chinese/English)
    "backend_url": None,                    # Optional: Custom API endpoint
    "max_debate_rounds": 3,                 # Bull/Bear debate iterations
    "enable_policy_analyst": True,          # A-share specific analysts
    "enable_hotmoney_analyst": True,
    "enable_lockup_analyst": True,
}
```

## Core API Usage

### Basic Analysis

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph

# Initialize with MiniMax
config = {
    "llm_provider": "minimax",
    "deep_think_llm": "MiniMax-M2.7",
    "quick_think_llm": "MiniMax-M2.7-highspeed",
    "output_language": "Chinese",
}

ta = TradingAgentsGraph(debug=True, config=config)

# Run analysis for stock 688017 on 2026-05-12
final_state, decision = ta.propagate("688017", "2026-05-12")

# Access decision
print(f"Signal: {decision['signal']}")          # BUY, HOLD, or SELL
print(f"Confidence: {decision['confidence']}")  # 0-100
print(f"Position: {decision['position_size']}")  # Suggested position size
print(f"Reasoning: {decision['reasoning']}")
```

### Using DeepSeek

```python
config = {
    "llm_provider": "deepseek",
    "deep_think_llm": "deepseek-chat",
    "quick_think_llm": "deepseek-chat",
    "output_language": "Chinese",
}

ta = TradingAgentsGraph(config=config)
final_state, decision = ta.propagate("600519", "2026-05-15")
```

### Using Anthropic with Kimi Backend

```python
config = {
    "llm_provider": "anthropic",
    "deep_think_llm": "claude-sonnet-4-6",
    "quick_think_llm": "claude-sonnet-4-6",
    "backend_url": "https://api.kimi.com/coding/",
    "output_language": "Chinese",
}

ta = TradingAgentsGraph(config=config)
final_state, decision = ta.propagate("000001", "2026-05-20")
```

### Accessing Individual Analyst Reports

```python
final_state, decision = ta.propagate("688017", "2026-05-12")

# Access 7 analyst reports
market_report = final_state["market_analyst_report"]
social_report = final_state["social_analyst_report"]
news_report = final_state["news_analyst_report"]
fundamentals_report = final_state["fundamentals_analyst_report"]
policy_report = final_state["policy_analyst_report"]
hotmoney_report = final_state["hotmoney_analyst_report"]
lockup_report = final_state["lockup_analyst_report"]

# Access debate and risk assessment
bull_research = final_state["bull_researcher_report"]
bear_research = final_state["bear_researcher_report"]
risk_assessment = final_state["risk_assessment"]
trading_plan = final_state["trading_plan"]
```

## CLI Commands

### Interactive Mode

```bash
# Launch interactive CLI
tradingagents

# CLI will prompt for:
# - Stock code (e.g., 688017, 600519)
# - Analysis date (YYYY-MM-DD)
# - LLM provider selection
```

### Direct Execution

```bash
# Analyze with specific parameters
tradingagents --stock 688017 --date 2026-05-12 --provider minimax

# Use different models
tradingagents --stock 600519 --date 2026-05-15 \
  --provider deepseek \
  --deep-model deepseek-chat \
  --quick-model deepseek-chat

# Enable debug output
tradingagents --stock 000001 --date 2026-05-20 --debug
```

### Web UI

```bash
# Launch Streamlit web interface
tradingagents-web

# Alternative
streamlit run web/app.py

# Access at http://localhost:8501
```

## Data Source Integration

### Available Data Tools

The framework provides these tools to analysts (all free, no API keys needed):

```python
# Market data (OHLCV, indicators)
get_stock_data(ticker, start_date, end_date)
get_indicators(ticker, start_date, end_date)

# Fundamental data
get_fundamentals(ticker)
get_balance_sheet(ticker)
get_cashflow(ticker)
get_income_statement(ticker)

# News and sentiment
get_news(ticker, start_date, end_date)
get_global_news(start_date, end_date)

# A-share specific
get_insider_transactions(ticker)  # Lockup releases, insider trading
# Dragon-Tiger list data available via get_news
```

### Data Provider Mapping

| Data Type | Provider | Protocol |
|-----------|----------|----------|
| OHLCV K-lines | mootdx | TCP 7709 |
| PE/PB/Market Cap | Tencent Finance | HTTP |
| Dragon-Tiger List | EastMoney | HTTP |
| Lockup Schedule | EastMoney | HTTP |
| Financial Statements | Sina Finance | HTTP |
| EPS Consensus | THS (10jqka) | HTTP |
| News Feed | CLS.cn | HTTP |
| Sector Classification | Baidu Finance | HTTP |

## Agent Pipeline Architecture

### 12-Stage Execution Flow

```python
# Stage 1-7: Analyst Reports
stages = [
    "market_analyst",      # Technical analysis
    "social_analyst",      # Social sentiment
    "news_analyst",        # News and events
    "fundamentals_analyst", # Financial statements
    "policy_analyst",      # Regulatory policy (A-share specific)
    "hotmoney_analyst",    # Dragon-Tiger list tracking (A-share specific)
    "lockup_analyst",      # Share lockup monitoring (A-share specific)
]

# Stage 8: Quality Gate
# Checks if analysts provided sufficient data

# Stage 9-10: Bull/Bear Debate
# Bull and Bear researchers debate up to N rounds

# Stage 11: Research Manager
# Deep-think LLM synthesizes debate into investment plan

# Stage 12: Trader + Risk Assessment
# Trader proposes execution plan
# 3 risk debaters (Aggressive, Conservative, Neutral) assess

# Stage 13: Portfolio Manager
# Deep-think LLM makes final BUY/HOLD/SELL decision
```

### Custom Analyst Configuration

```python
# Disable A-share specific analysts
config = {
    "llm_provider": "minimax",
    "deep_think_llm": "MiniMax-M2.7",
    "quick_think_llm": "MiniMax-M2.7-highspeed",
    "output_language": "Chinese",
    "enable_policy_analyst": False,
    "enable_hotmoney_analyst": False,
    "enable_lockup_analyst": False,
}

ta = TradingAgentsGraph(config=config)
```

## A-Share Trading Constraints

The framework automatically applies A-share market rules:

```python
# T+1 Settlement
# Cannot sell shares bought today

# Price Limits
# ST stocks: ±5%
# Regular stocks: ±10% (or ±20% for ChiNext/STAR)

# Minimum Lot Size
# 100 shares (1 lot)
# Must trade in multiples of 100

# Trading Hours
# Morning: 09:30-11:30
# Afternoon: 13:00-15:00
# Call auction: 09:15-09:25, 14:57-15:00
```

These constraints are built into the Trader agent's decision logic.

## Common Patterns

### Batch Analysis

```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from datetime import datetime, timedelta

config = {
    "llm_provider": "minimax",
    "deep_think_llm": "MiniMax-M2.7",
    "quick_think_llm": "MiniMax-M2.7-highspeed",
    "output_language": "Chinese",
}

ta = TradingAgentsGraph(config=config)

# Analyze portfolio of stocks
stocks = ["600519", "000858", "600036", "601318"]
date = "2026-05-15"

results = {}
for ticker in stocks:
    try:
        final_state, decision = ta.propagate(ticker, date)
        results[ticker] = decision
        print(f"{ticker}: {decision['signal']} (confidence: {decision['confidence']})")
    except Exception as e:
        print(f"Error analyzing {ticker}: {e}")
        continue
```

### Time Series Analysis

```python
from datetime import datetime, timedelta

ta = TradingAgentsGraph(config=config)
ticker = "688017"

# Analyze over 5 trading days
base_date = datetime(2026, 5, 10)
signals = []

for i in range(5):
    analysis_date = (base_date + timedelta(days=i)).strftime("%Y-%m-%d")
    try:
        final_state, decision = ta.propagate(ticker, analysis_date)
        signals.append({
            "date": analysis_date,
            "signal": decision["signal"],
            "confidence": decision["confidence"],
        })
    except Exception as e:
        print(f"Error on {analysis_date}: {e}")
        continue

# Track signal consistency
print(f"Signal history for {ticker}:")
for s in signals:
    print(f"{s['date']}: {s['signal']} ({s['confidence']}%)")
```

### Custom LLM Backend

```python
# Use local Ollama instance
config = {
    "llm_provider": "ollama",
    "deep_think_llm": "qwen2.5:32b",
    "quick_think_llm": "qwen2.5:14b",
    "backend_url": "http://localhost:11434",
    "output_language": "Chinese",
}

ta = TradingAgentsGraph(config=config)
final_state, decision = ta.propagate("600519", "2026-05-15")
```

### Extracting Structured Data

```python
final_state, decision = ta.propagate("688017", "2026-05-12")

# Extract key metrics from analyst reports
def extract_pe_ratio(fundamentals_report):
    # Parse PE from report text
    import re
    match = re.search(r'PE.*?(\d+\.\d+)', fundamentals_report)
    return float(match.group(1)) if match else None

def extract_lockup_events(lockup_report):
    # Parse lockup schedule
    events = []
    lines = lockup_report.split('\n')
    for line in lines:
        if '解禁' in line:
            events.append(line.strip())
    return events

pe_ratio = extract_pe_ratio(final_state["fundamentals_analyst_report"])
lockup_events = extract_lockup_events(final_state["lockup_analyst_report"])

print(f"PE Ratio: {pe_ratio}")
print(f"Upcoming lockups: {lockup_events}")
```

## Troubleshooting

### API Key Issues

```python
# Verify env vars are loaded
import os
from dotenv import load_dotenv

load_dotenv()
print(f"MINIMAX_API_KEY exists: {bool(os.getenv('MINIMAX_API_KEY'))}")

# If still failing, pass key directly (not recommended for production)
config = {
    "llm_provider": "minimax",
    "api_key": "sk-your-key-here",  # Override env var
    "deep_think_llm": "MiniMax-M2.7",
    "quick_think_llm": "MiniMax-M2.7-highspeed",
}
```

### Data Source Failures

```python
# Test data connectivity
from tradingagents.tools.astock_tools import get_stock_data

try:
    data = get_stock_data("688017", "2026-04-01", "2026-05-01")
    print(f"Successfully fetched {len(data)} rows")
except Exception as e:
    print(f"Data fetch failed: {e}")
    # Fallback: Check network, try different date range
```

### Invalid Stock Code

```python
# Validate A-share ticker format
import re

def validate_astock_ticker(ticker):
    # A-share codes: 6 digits
    # Shanghai: 600xxx, 601xxx, 603xxx, 688xxx (STAR)
    # Shenzhen: 000xxx, 001xxx, 002xxx, 003xxx, 300xxx (ChiNext)
    pattern = r'^(600|601|603|688|000|001|002|003|300)\d{3}$'
    return bool(re.match(pattern, ticker))

ticker = "688017"
if not validate_astock_ticker(ticker):
    print(f"Invalid ticker: {ticker}")
```

### Rate Limiting

```python
# Add delays for batch analysis
import time

ta = TradingAgentsGraph(config=config)
tickers = ["600519", "000858", "600036"]

for ticker in tickers:
    final_state, decision = ta.propagate(ticker, "2026-05-15")
    print(f"{ticker}: {decision['signal']}")
    time.sleep(5)  # 5 second delay between requests
```

### Debug Mode

```python
# Enable verbose logging
ta = TradingAgentsGraph(debug=True, config=config)
final_state, decision = ta.propagate("688017", "2026-05-12")

# Inspect intermediate states
print("Market Analyst Report:")
print(final_state["market_analyst_report"])
print("\nBull Research:")
print(final_state["bull_researcher_report"])
print("\nBear Research:")
print(final_state["bear_researcher_report"])
```

### LLM Response Parsing Errors

```python
# If LLM returns malformed JSON, enable retry logic
config = {
    "llm_provider": "minimax",
    "deep_think_llm": "MiniMax-M2.7",
    "quick_think_llm": "MiniMax-M2.7-highspeed",
    "max_retries": 3,  # Retry up to 3 times on parse errors
    "temperature": 0.7,  # Lower temperature for more consistent formatting
}

ta = TradingAgentsGraph(config=config)
```

## Project Structure

```
tradingagents-astock/
├── tradingagents/
│   ├── graph/
│   │   ├── trading_graph.py      # Main TradingAgentsGraph class
│   │   └── nodes.py               # Individual agent node implementations
│   ├── tools/
│   │   ├── astock_tools.py        # A-share data fetching tools
│   │   └── tool_registry.py       # Tool registration system
│   ├── llm/
│   │   ├── llm_factory.py         # LLM provider abstraction
│   │   └── providers/             # Provider-specific implementations
│   └── agents/
│       ├── analysts/              # 7 analyst agent prompts
│       ├── researchers/           # Bull/Bear researchers
│       ├── risk/                  # 3 risk debaters
│       └── portfolio_manager/     # Final decision maker
├── web/
│   └── app.py                     # Streamlit UI
├── .env.example                   # Environment variable template
└── README.md
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Start with debug=True** to understand the pipeline
3. **Use MiniMax or DeepSeek** for cost-effective China-based inference
4. **Check data availability** before running batch analyses (market holidays, weekends)
5. **Monitor LLM costs** — each analysis requires 30-50 API calls
6. **Validate stock codes** before passing to `propagate()`
7. **Set appropriate `max_debate_rounds`** — more rounds = higher cost but potentially better decisions
8. **Use quick_think_llm** for analysts and researchers, reserve deep_think_llm for final decisions
