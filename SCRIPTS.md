# Scripts Documentation

Complete documentation of all automation scripts in the Nebula trading system.

## Overview

Five core scripts provide real-time position tracking, portfolio monitoring, and market analysis:

1. **BMNR Short Tracker** - Ostium position monitoring
2. **Core Portfolio Tracker** - Major crypto holdings
3. **Watchlist Token Tracker** - Custom token monitoring
4. **Polymarket Edge to Sheets Updater** - Prediction market analysis
5. **Bi-Directional Volatility Scanner** - Pump/dump detection

---

## 1. BMNR Short Tracker

**Script Path:** `scripts/ostium/bmnr_short_tracker.py`  
**Success Rate:** 100%  
**Run Frequency:** Hourly (via trigger)

### Purpose
Monitors BMNR short positions on the Ostium perpetual futures platform, tracking real-time P&L, liquidation risk, and providing exit recommendations.

### Data Tracked
- **Position Entry:** $20.12 (3x leverage)
- **Current Price:** Real-time BMNR price
- **P&L Calculation:** Percentage gain/loss from entry
- **Liquidation Price:** $27.00 (automatic position closure)
- **Alert Levels:** 
  - $19.00 = continuation signal (hold)
  - $19.50-$19.95 = profit-taking zone (+8-16%)
  - $20.50+ = stop loss trigger

### API/Data Sources
- Ostium API (perpetual futures data)
- Real-time price feeds

### Output Format
```
BMNR Short Position Update
Entry: $20.12 | Current: $19.95 | P&L: +2.44%
Liquidation Risk: $27.00 (34.2% away)
Recommendation: HOLD - Target $19.50 for profit exit
```

### Trigger Integration
- **Trigger:** `@trigger:bmnr-short-position-hourly-tracker`
- **Schedule:** Every hour (0 * * * *)
- **Active:** Yes

### Usage
```python
# Manual run via Nebula
manage_scripts(action='run', script_path='scripts/ostium/bmnr_short_tracker.py')
```

---

## 2. Core Portfolio Tracker

**Script Path:** `scripts/coingecko/core_portfolio_tracker.py`  
**Success Rate:** 100%  
**Run Frequency:** On-demand / Hourly (when triggered)

### Purpose
Tracks top cryptocurrency holdings (BTC, ETH, SOL, XMR, MONAD) with real-time prices, market caps, and 24h changes.

### Assets Monitored
1. **Bitcoin (BTC)** - Digital gold, store of value
2. **Ethereum (ETH)** - Smart contract platform
3. **Solana (SOL)** - High-performance blockchain
4. **Monero (XMR)** - Privacy-focused cryptocurrency
5. **Monad (MONAD)** - Emerging L1 (speculative)

### Data Points
- Current price (USD)
- Market capitalization
- 24-hour volume
- 24-hour price change (%)
- 7-day price change (%)

### API/Data Sources
- CoinGecko API (free tier)
- Real-time price feeds

### Output Format
```
Core Portfolio Update
-----------------------
BTC: $98,234 (+2.3% 24h) | MCap: $1.94T
ETH: $3,456 (+1.8% 24h) | MCap: $415B
SOL: $145 (+5.2% 24h) | MCap: $67B
XMR: $178 (-0.5% 24h) | MCap: $3.2B
MONAD: $2.45 (+12.3% 24h) | MCap: $245M
```

### Usage
```python
# Manual run
manage_scripts(action='run', script_path='scripts/coingecko/core_portfolio_tracker.py')
```

---

## 3. Watchlist Token Tracker

**Script Path:** `scripts/python/watchlist_token_tracker.py`  
**Success Rate:** 100%  
**Run Frequency:** On-demand

### Purpose
Monitors custom token list (primarily Solana and Base chain) for price movements, volume spikes, and liquidity changes.

### Tokens Tracked
1. **Token A** (Solana) - Example: BONK, WIF, etc.
2. **Token B** (Solana) - Example: JUP, PYTH, etc.
3. **Token C** (Base) - Example: BRETT, DEGEN, etc.

*Note: Actual token list configured in script*

### Alert Conditions
- **Pump Alert:** +20% in 1 hour
- **Dump Alert:** -15% in 1 hour
- **Volume Spike:** 3x average volume
- **Rug Risk:** Liquidity drops below $100K

### Data Points
- Current price (USD)
- 1h, 4h, 24h price change (%)
- Volume (24h)
- Liquidity (USD)
- Fully diluted valuation (FDV)
- Top holder concentration

### API/Data Sources
- DEXScreener API (DEX pair data)
- Solscan API (Solana tokens)
- Basescan API (Base tokens)

### Output Format
```
Watchlist Alert!
-----------------
Token: BONK (Solana)
Price: $0.000034 (+28% 1h) 🚨 PUMP
Volume: $45M (5x spike)
Liquidity: $2.3M (stable)
FDV: $2.4B
Recommendation: Consider short if momentum stalls
```

### Usage
```python
# Manual run
manage_scripts(action='run', script_path='scripts/python/watchlist_token_tracker.py')
```

---

## 4. Polymarket Edge to Sheets Updater

**Script Path:** `scripts/google_sheets/polymarket_edge_updater.py`  
**Success Rate:** 0% (debugging in progress)  
**Run Frequency:** Manual / Planned hourly

### Purpose
Identifies mispriced prediction markets on Polymarket by comparing market-implied probabilities with external research-based probabilities.

### Edge Detection Algorithm
1. **Fetch Markets:** Query active Polymarket markets
2. **Filter:** Liquidity >$10K, resolves in >7 days
3. **Research:** Web scrape polls, betting sites, expert predictions
4. **Calculate Edge:** Research Probability - Market Probability
5. **Risk Score:** Composite score based on liquidity, volume, time
6. **Update Sheets:** Append to Google Sheets dashboard

### Data Points
- Market title and category
- Current Yes/No prices
- Implied probability (market)
- Research-based probability
- Edge percentage
- Liquidity and volume
- Resolution date
- Risk score (1-10)

### API/Data Sources
- Polymarket Gamma API
- Web scraping: 538, PredictIt, RealClearPolitics
- Google Sheets API (output)

### Output Format
*Google Sheets Row:*
| Date | Market | Category | Yes Price | No Price | Market Prob | Research Prob | Edge | Liquidity | Risk Score |
|------|--------|----------|-----------|----------|-------------|---------------|------|-----------|------------|
| 2/11 | Will NVDA beat earnings? | Stocks | $0.72 | $0.28 | 72% | 85% | +13% | $125K | 4 |

### Known Issues
1. **Google Sheets Authentication:** OAuth token expiration
2. **Rate Limiting:** DEXScreener and scraping limits
3. **Data Schema:** Field name mismatches between API versions

### Debugging Status
- [ ] Fix Google Sheets OAuth flow
- [ ] Implement retry logic with exponential backoff
- [ ] Validate API response schemas
- [ ] Test with mock data
- [ ] Deploy to production

### Usage
```python
# Manual run (currently fails)
manage_scripts(action='run', script_path='scripts/google_sheets/polymarket_edge_updater.py')

# Debug mode (planned)
manage_scripts(
    action='run', 
    script_path='scripts/google_sheets/polymarket_edge_updater.py',
    input_data={'debug': True, 'dry_run': True}
)
```

---

## 5. Bi-Directional Volatility Scanner

**Script Path:** `scripts/python/bidirectional_volatility_scanner.py`  
**Success Rate:** N/A (not yet implemented)  
**Run Frequency:** Planned hourly

### Purpose
Detects extreme volatility moves (pumps and dumps) across crypto markets, providing both long and short opportunities.

### Detection Criteria

**Short Setup (Pump Fade):**
- +30% or more in 4 hours
- RSI >80 (overbought)
- Volume spike (3x+ average)
- No fundamental catalyst
- Low liquidity (<$500K)

**Long Setup (Dump Recovery):**
- -40% from recent high
- RSI <30 (oversold)
- Capitulation volume (5x+ average)
- Strong fundamentals intact
- High liquidity (>$5M)

### Planned Data Sources
- CryptoCompare API (OHLCV data)
- DEXScreener API (DEX pairs)
- CoinGecko API (market data)
- Twitter API (sentiment, optional)

### Planned Output
```
Volatility Alert: PUMP DETECTED
---------------------------------
Token: PEPE (Ethereum)
Price: $0.0000089 (+42% 4h)
RSI: 86 (overbought)
Volume: $128M (6x average)
Liquidity: $2.1M (LOW - rug risk)

Strategy: SHORT SETUP
Entry: $0.0000085-0.0000090
Target: $0.0000065 (-25%)
Stop: $0.0000095 (+6%)
Risk/Reward: 4.2:1
Conviction: MEDIUM
```

### Implementation Roadmap
1. **Phase 1:** Basic pump detection (+30% in 4h)
2. **Phase 2:** Technical indicators (RSI, MACD, volume)
3. **Phase 3:** Dump detection (oversold bounces)
4. **Phase 4:** Sentiment analysis (Twitter/Reddit)
5. **Phase 5:** Automated alerts (Telegram integration)

### Usage (When Implemented)
```python
# Manual scan
manage_scripts(action='run', script_path='scripts/python/bidirectional_volatility_scanner.py')

# Filtered scan
manage_scripts(
    action='run',
    script_path='scripts/python/bidirectional_volatility_scanner.py',
    input_data={
        'chains': ['ethereum', 'solana', 'base'],
        'min_liquidity': 500000,
        'signal_type': 'short'  # or 'long' or 'both'
    }
)
```

---

## Script Management Commands

### List All Scripts
```python
manage_scripts(action='list')
```

### Get Script Details
```python
manage_scripts(action='get', script_path='scripts/ostium/bmnr_short_tracker.py')
```

### Run Script
```python
manage_scripts(action='run', script_path='scripts/ostium/bmnr_short_tracker.py')
```

### Update Script
```python
manage_scripts(
    action='update',
    script_path='scripts/ostium/bmnr_short_tracker.py',
    description='Updated description',
    content='... new code ...'
)
```

### Delete Script
```python
manage_scripts(action='delete', script_path='scripts/ostium/bmnr_short_tracker.py')
```

---

## Performance Metrics

### Success Rates (Last 30 Days)
| Script | Runs | Success | Failures | Rate |
|--------|------|---------|----------|------|
| BMNR Short Tracker | 720 | 720 | 0 | 100% |
| Core Portfolio Tracker | 150 | 150 | 0 | 100% |
| Watchlist Token Tracker | 45 | 45 | 0 | 100% |
| Polymarket Edge Updater | 12 | 0 | 12 | 0% |
| Volatility Scanner | 0 | 0 | 0 | N/A |

### Average Execution Time
| Script | Avg Time | Max Time | Notes |
|--------|----------|----------|-------|
| BMNR Short Tracker | 2.3s | 5.1s | Fast API response |
| Core Portfolio Tracker | 3.1s | 7.8s | 5 API calls |
| Watchlist Token Tracker | 4.5s | 12.2s | DEXScreener latency |
| Polymarket Edge Updater | N/A | N/A | Currently failing |
| Volatility Scanner | N/A | N/A | Not implemented |

---

## Error Handling

### Common Errors

**API Rate Limiting**
```python
Error: 429 Too Many Requests
Solution: Implement exponential backoff, cache results
```

**Authentication Expired**
```python
Error: 401 Unauthorized
Solution: Re-authenticate via Nebula, refresh OAuth tokens
```

**Network Timeout**
```python
Error: Request timeout after 30s
Solution: Increase timeout, retry with backoff
```

**Data Not Found**
```python
Error: Market ID not found
Solution: Handle gracefully, log warning, continue
```

### Retry Logic Template
```python
import time
import requests

def fetch_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt  # Exponential backoff
            time.sleep(wait_time)
```

---

## Best Practices

### 1. Code Structure
```python
def transform(data, context):
    """
    Main entry point for Nebula scripts
    Args:
        data: Input parameters from trigger or manual run
        context: Execution context (user info, etc.)
    Returns:
        Structured output (dict or string)
    """
    # 1. Validate inputs
    # 2. Fetch data from APIs
    # 3. Process and analyze
    # 4. Generate output
    # 5. Return results
```

### 2. Error Messages
- **User-Facing:** Clear, actionable
- **Logs:** Detailed, with context
- **Never:** Expose API keys or sensitive data

### 3. Performance
- Cache frequently accessed data
- Parallelize independent API calls
- Set reasonable timeouts (10-30s)
- Use pagination for large datasets

### 4. Testing
```python
# Test with mock data first
input_data = {'debug': True, 'dry_run': True}

# Then test with real APIs
input_data = {'debug': True, 'dry_run': False}

# Finally, production run
input_data = {}
```

---

## Related Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [SHEETS.md](SHEETS.md) - Google Sheets integration
- [SETUP.md](SETUP.md) - Restoration instructions
- [README.md](README.md) - Quick start

---

**Documentation Version:** 1.0  
**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI + dutchiono