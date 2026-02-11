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
**Run Frequency:** On-demand / Hourly

### Purpose
Tracks top-tier cryptocurrency holdings with real-time price, market cap, 24h volume, and percentage changes.

### Assets Tracked
1. **BTC** (Bitcoin)
2. **ETH** (Ethereum)
3. **SOL** (Solana)
4. **XMR** (Monero)
5. **MONAD** (Monad)

### Data Points
- Current Price (USD)
- Market Cap
- 24h Trading Volume
- 24h Price Change (%)
- 7d Price Change (%)

### API/Data Sources
- **CoinGecko API**
  - Endpoint: `/coins/markets`
  - Rate Limit: 10-50 calls/minute (free tier)

### Output Format
```
Core Portfolio Update - 2026-02-11 01:40 EST

BTC: $47,234 | MCap: $927B | Vol: $28.4B | 24h: +2.1% | 7d: +8.4%
ETH: $2,543 | MCap: $306B | Vol: $14.2B | 24h: +1.8% | 7d: +6.2%
SOL: $102.45 | MCap: $45.3B | Vol: $2.1B | 24h: +4.3% | 7d: +12.1%
XMR: $168.34 | MCap: $3.1B | Vol: $87M | 24h: -0.5% | 7d: +3.2%
MONAD: $12.84 | MCap: $1.2B | Vol: $145M | 24h: +15.2% | 7d: +42.8%

Portfolio Summary: +3.8% (24h) | +11.2% (7d)
```

### Usage
```python
# Run via Nebula
manage_scripts(action='run', script_path='scripts/coingecko/core_portfolio_tracker.py')
```

---

## 3. Watchlist Token Tracker

**Script Path:** `scripts/python/watchlist_token_tracker.py`  
**Success Rate:** 100%  
**Run Frequency:** On-demand

### Purpose
Monitors custom tokens across multiple chains (Solana, Base) with real-time price tracking and volatility alerts.

### Tokens Tracked
1. **Token 1** (Solana) - Custom address
2. **Token 2** (Solana) - Custom address  
3. **Token 3** (Base) - Custom address

### Data Points
- Real-time Price
- 1h, 24h, 7d Price Changes
- Volume (24h)
- Liquidity Pool Status
- DEX Trading Activity

### API/Data Sources
- **DEXScreener API**
  - Endpoint: `/latest/dex/tokens/{address}`
  - Chain support: Solana, Base
- **Solscan API** (Solana-specific)
- **Basescan API** (Base-specific)

### Output Format
```
Watchlist Update - 2026-02-11 01:40 EST

[Solana Token 1]
Price: $0.00234 | 1h: +5.2% | 24h: +18.4% | 7d: +42.1%
Volume: $284K | Liquidity: $1.2M
Status: TRENDING - High volume spike detected

[Solana Token 2]
Price: $0.156 | 1h: -2.1% | 24h: -8.3% | 7d: -15.2%
Volume: $48K | Liquidity: $890K
Status: DECLINING - Low volume, possible exit

[Base Token 3]
Price: $1.84 | 1h: +0.8% | 24h: +3.2% | 7d: +9.4%
Volume: $524K | Liquidity: $3.4M
Status: STABLE - Healthy trading range
```

### Alert Thresholds
- **Pump Alert:** +20% in 1h
- **Dump Alert:** -15% in 1h
- **Volume Spike:** 3x average volume
- **Liquidity Warning:** <$100K (rug risk)

### Usage
```python
# Run via Nebula
manage_scripts(action='run', script_path='scripts/python/watchlist_token_tracker.py')
```

---

## 4. Polymarket Edge to Sheets Updater

**Script Path:** `scripts/google_sheets/polymarket_edge_updater.py`  
**Success Rate:** 0% (needs debugging)  
**Run Frequency:** Manual / On-demand

### Purpose
Identifies mispriced prediction markets on Polymarket by comparing implied probabilities against external data sources, then populates a Google Sheets dashboard.

### Edge Detection Process
1. **Fetch Markets:** Query Polymarket Gamma API for active markets
2. **External Research:** Web scrape polls, news, statistical models
3. **Calculate Edge:** Compare market odds vs. research-based probability
4. **Rank Opportunities:** Sort by edge size and liquidity
5. **Update Sheet:** Populate Edge Candidates sheet with findings

### Data Points
- Market Title & Description
- Current Odds (Yes/No)
- Implied Probability
- Research-Based Probability
- Edge Percentage
- Liquidity & Volume
- Time to Resolution
- Risk Score

### API/Data Sources
- **Polymarket Gamma API**
  - `/markets` - Market discovery
  - `/prices` - Real-time odds
  - `/events` - Event grouping
- **Web Scraping:** Polls, news, expert predictions
- **Google Sheets API:** Automated updates

### Google Sheets Integration
See [SHEETS.md](SHEETS.md) for detailed sheet structure.

### Known Issues
- Script currently failing (0% success rate)
- Likely issues:
  - Google Sheets authentication
  - API rate limiting
  - Data formatting mismatch

### Planned Fixes
1. Debug Sheets API authentication
2. Add retry logic for rate limits
3. Validate data schema before insertion
4. Add error logging

### Usage
```python
# Run via Nebula (currently failing)
manage_scripts(action='run', script_path='scripts/google_sheets/polymarket_edge_updater.py')
```

---

## 5. Bi-Directional Volatility Scanner

**Script Path:** `scripts/python/bidirectional_volatility_scanner.py`  
**Success Rate:** 0% (needs implementation)  
**Run Frequency:** Planned hourly

### Purpose
Detects BOTH parabolic pumps (short opportunities) and falling knives (potential long entries) across crypto markets.

### Detection Criteria

#### Pump Detection (Short Setup)
- +30% in 4 hours
- Volume spike >5x average
- RSI >80 (overbought)
- Liquidity sufficient for exit
- No major catalyst (news-driven pumps excluded)

#### Dump Detection (Long Setup)  
- -40% from recent high
- RSI <30 (oversold)
- Volume declining (panic selling exhaustion)
- Fundamental value intact (not rug pull)
- Potential reversal patterns

### Data Sources
- **CryptoCompare API:** OHLCV data
- **DEXScreener API:** DEX pair monitoring
- **CoinGecko API:** Market cap, volume
- **Sentiment APIs:** Social volume tracking

### Output Format
```
Volatility Scanner - 2026-02-11 01:40 EST

SHORT OPPORTUNITIES (Parabolic Pumps)
1. TOKEN_XYZ: +47% (4h) | RSI: 86 | Vol: 8.2x | Liq: $2.4M
   Recommendation: SHORT at $0.45 | Target: $0.32 | Stop: $0.48

2. MEMECOIN_ABC: +68% (4h) | RSI: 92 | Vol: 12.4x | Liq: $890K
   Recommendation: SHORT at $0.0023 | Target: $0.0016 | Stop: $0.0025

LONG OPPORTUNITIES (Falling Knives)
1. SOLID_TOKEN: -38% from high | RSI: 24 | Vol: declining | MCap: $45M
   Recommendation: LONG at $1.82 | Target: $2.40 | Stop: $1.65
   
2. DEFI_BLUE: -42% from high | RSI: 28 | Vol: capitulation | MCap: $120M
   Recommendation: LONG at $8.45 | Target: $12.00 | Stop: $7.80
```

### Alert Integration
- Telegram notifications for high-conviction setups
- Google Sheets logging of all signals
- Backtest results tracking

### Status
**NOT YET IMPLEMENTED**  
- Script exists but not functional
- Needs API integration setup
- Requires backtesting before live use

### Planned Implementation
1. Build OHLCV data pipeline
2. Implement technical indicators (RSI, volume)
3. Create scoring algorithm
4. Add risk filters (liquidity, market cap)
5. Test on historical data
6. Deploy with alerts

---

## Script Management

### Listing All Scripts
```python
manage_scripts(action='list')
```

### Running a Script
```python
manage_scripts(action='run', script_path='scripts/ostium/bmnr_short_tracker.py')
```

### Updating a Script
```python
manage_scripts(
    action='update',
    script_path='scripts/ostium/bmnr_short_tracker.py',
    content='<new_code>'
)
```

### Viewing Script Details
```python
manage_scripts(action='get', script_path='scripts/ostium/bmnr_short_tracker.py')
```

---

## Trigger Integration

Scripts can be automated via cron triggers:

### Active Triggers
1. **BMNR Hourly Tracker**
   - Trigger: `@trigger:bmnr-short-position-hourly-tracker`
   - Schedule: `0 * * * *` (every hour)
   - Script: BMNR Short Tracker

### Creating New Triggers
```python
manage_triggers(
    action='create',
    name='Core Portfolio Hourly Update',
    description='Track BTC, ETH, SOL, XMR, MONAD prices every hour',
    trigger_type='cron',
    cron_expression='0 * * * *',
    recipe='path/to/task_recipe.md'
)
```

---

## Troubleshooting

### Script Failing?

1. **Check Success Rate**
   ```python
   manage_scripts(action='list')
   # Look for success rate < 100%
   ```

2. **View Error Logs**
   ```python
   manage_scripts(action='get', script_path='path/to/script.py')
   # Check recent execution logs
   ```

3. **Common Issues**
   - API authentication expired
   - Rate limiting (429 errors)
   - Data format changes
   - Missing dependencies

4. **Quick Fixes**
   - Re-authenticate APIs via Nebula
   - Add retry logic with backoff
   - Update data parsing logic
   - Install missing packages

---

## Adding New Scripts

### Script Template
```python
def transform(data, context):
    """
    Main script function called by Nebula
    
    Args:
        data (dict): Input parameters
        context (dict): Execution context (time, user, etc.)
    
    Returns:
        dict: Script output with status and results
    """
    try:
        # Your script logic here
        result = {
            'status': 'success',
            'data': {},
            'message': 'Script executed successfully'
        }
        return result
    except Exception as e:
        return {
            'status': 'error',
            'message': str(e)
        }
```

### Creating the Script
```python
manage_scripts(
    action='create',
    name='New Script Name',
    description='What the script does',
    content='<script_code>',
    app_slugs=['relevant', 'integrations']
)
```

---

## Maintenance Schedule

### Daily
- Review BMNR position tracker alerts
- Check core portfolio performance
- Monitor watchlist for entry/exit signals

### Weekly
- Audit script success rates
- Review and optimize slow scripts
- Update API credentials if needed

### Monthly
- Backtest volatility scanner signals
- Refine edge detection algorithms
- Archive old logs and data

---

## Related Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) - Full system overview
- [SHEETS.md](SHEETS.md) - Google Sheets integration
- [SETUP.md](SETUP.md) - Rapid restoration guide

---

**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI System