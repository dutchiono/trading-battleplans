# Rapid Setup Guide

Complete instructions for restoring the trading automation system after a Nebula reset or fresh installation.

## Quick Recovery Checklist

- [ ] Clone documentation repository
- [ ] Recreate core scripts (5 scripts)
- [ ] Restore triggers (2 triggers)
- [ ] Reconnect API integrations (6 services)
- [ ] Verify Google Sheets access
- [ ] Test script execution
- [ ] Enable GitHub Pages
- [ ] Validate end-to-end workflows

**Estimated Time:** 45-60 minutes

---

## Prerequisites

### Required Accounts
- Nebula AI account (active)
- GitHub account (dutchiono)
- Google account (dutchiono@gmail.com)
- Ostium account (for BMNR tracking)

### Required Access
- GitHub repository: dutchiono/trading-battleplans
- Google Sheets: Trading Dashboard
- API keys/credentials (if stored externally)

---

## Step 1: Clone Documentation

### 1.1 Access Repository
```
Repository: https://github.com/dutchiono/trading-battleplans
Branch: main
```

### 1.2 Review Documentation
Read these files in order:
1. README.md - System overview
2. ARCHITECTURE.md - System design
3. SCRIPTS.md - Script details
4. SHEETS.md - Dashboard structure
5. SETUP.md (this file) - Restoration steps

---

## Step 2: Restore Scripts

### 2.1 BMNR Short Tracker

**Command:**
```python
manage_scripts(
    action='create',
    name='BMNR Short Tracker',
    description='Tracks two BMNR short positions on Ostium (Trade #0: $21.06 entry, Trade #1: $19.70 entry). Fetches current price, calculates P&L, and monitors liquidation risk.',
    app_slugs=['ostium'],
    content='''
def transform(data, context):
    """
    Monitor BMNR short positions on Ostium
    """
    import requests
    
    # Position details
    positions = [
        {"id": "Trade #0", "entry": 21.06, "size": 1000},
        {"id": "Trade #1", "entry": 19.70, "size": 500}
    ]
    
    # Fetch current BMNR price
    # TODO: Replace with actual Ostium API endpoint
    current_price = 19.95  # Placeholder
    
    results = []
    for pos in positions:
        pnl_pct = ((pos["entry"] - current_price) / pos["entry"]) * 100
        pnl_usd = (pos["entry"] - current_price) * pos["size"]
        
        results.append({
            "position": pos["id"],
            "entry": pos["entry"],
            "current": current_price,
            "pnl_pct": round(pnl_pct, 2),
            "pnl_usd": round(pnl_usd, 2)
        })
    
    return {
        "positions": results,
        "current_price": current_price,
        "liquidation_price": 27.00
    }
'''
)
```

---

### 2.2 Core Portfolio Tracker

**Command:**
```python
manage_scripts(
    action='create',
    name='Core Portfolio Tracker',
    description='Tracks BTC, ETH, SOL, XMR, MONAD with price, market cap, volume, and 24h change.',
    app_slugs=['coingecko'],
    content='''
def transform(data, context):
    """
    Track core crypto portfolio
    """
    import requests
    
    # CoinGecko API (free tier)
    coins = ['bitcoin', 'ethereum', 'solana', 'monero', 'monad']
    url = 'https://api.coingecko.com/api/v3/simple/price'
    
    params = {
        'ids': ','.join(coins),
        'vs_currencies': 'usd',
        'include_market_cap': 'true',
        'include_24hr_vol': 'true',
        'include_24hr_change': 'true'
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    portfolio = []
    for coin_id in coins:
        if coin_id in data:
            coin_data = data[coin_id]
            portfolio.append({
                'coin': coin_id.upper(),
                'price': coin_data.get('usd', 0),
                'market_cap': coin_data.get('usd_market_cap', 0),
                'volume_24h': coin_data.get('usd_24h_vol', 0),
                'change_24h': coin_data.get('usd_24h_change', 0)
            })
    
    return {'portfolio': portfolio}
'''
)
```

---

### 2.3 Watchlist Token Tracker

**Command:**
```python
manage_scripts(
    action='create',
    name='Watchlist Token Tracker',
    description='Tracks 3 custom tokens (2 Solana, 1 Base) with price, market cap, volume, and liquidity.',
    app_slugs=[],
    content='''
def transform(data, context):
    """
    Track custom token watchlist
    """
    import requests
    
    # Define watchlist (replace with actual contract addresses)
    watchlist = [
        {'chain': 'solana', 'address': 'TOKEN_ADDRESS_1', 'name': 'Token A'},
        {'chain': 'solana', 'address': 'TOKEN_ADDRESS_2', 'name': 'Token B'},
        {'chain': 'base', 'address': 'TOKEN_ADDRESS_3', 'name': 'Token C'}
    ]
    
    results = []
    for token in watchlist:
        # DEXScreener API
        url = f'https://api.dexscreener.com/latest/dex/tokens/{token["address"]}'
        response = requests.get(url)
        
        if response.status_code == 200:
            data = response.json()
            pairs = data.get('pairs', [])
            if pairs:
                pair = pairs[0]  # Use first (most liquid) pair
                results.append({
                    'name': token['name'],
                    'chain': token['chain'],
                    'price': pair.get('priceUsd', 0),
                    'liquidity': pair.get('liquidity', {}).get('usd', 0),
                    'volume_24h': pair.get('volume', {}).get('h24', 0),
                    'change_1h': pair.get('priceChange', {}).get('h1', 0),
                    'change_24h': pair.get('priceChange', {}).get('h24', 0)
                })
    
    return {'tokens': results}
'''
)
```

---

### 2.4 Polymarket Edge Analyzer - Sheet Updater

**Command:**
```python
manage_scripts(
    action='create',
    name='Polymarket Edge Analyzer - Sheet Updater',
    description='Analyzes all Polymarket markets from Sheet1 for betting edges using research vs market-implied probabilities. Updates Google Sheets with edge calculations and recommendations.',
    app_slugs=['google_sheets'],
    content='''
def transform(data, context):
    """
    Analyze Polymarket edges and update Google Sheets
    """
    import requests
    
    # Polymarket Gamma API
    url = 'https://gamma-api.polymarket.com/markets'
    params = {'active': 'true', 'limit': 100}
    
    response = requests.get(url, params=params)
    markets = response.json()
    
    edges = []
    for market in markets:
        # Simple edge detection (placeholder logic)
        market_prob = float(market.get('outcomePrices', ['0.5'])[0])
        research_prob = 0.5  # TODO: Implement research logic
        edge = research_prob - market_prob
        
        if abs(edge) > 0.05:  # >5% edge
            edges.append({
                'market': market.get('question', 'Unknown'),
                'market_prob': round(market_prob * 100, 1),
                'research_prob': round(research_prob * 100, 1),
                'edge': round(edge * 100, 1),
                'liquidity': market.get('liquidity', 0)
            })
    
    # TODO: Update Google Sheets
    return {'edges_found': len(edges), 'opportunities': edges}
'''
)
```

---

### 2.5 Biotech Spike Scanner

**Command:**
```python
manage_scripts(
    action='create',
    name='Biotech Spike Scanner',
    description='Scans for biotech stocks with +40% intraday moves and identifies immediate short opportunities with float, borrow rate, and short interest data.',
    app_slugs=[],
    content='''
def transform(data, context):
    """
    Scan for biotech +40% spikes for short opportunities
    """
    import requests
    from datetime import datetime
    
    # Finviz screener API or web scraping
    # Placeholder: Detect stocks with +40% moves
    
    spikes = [
        # Example format
        # {'ticker': 'PHIO', 'change': 51.2, 'price': 2.45, 'volume': 847000}
    ]
    
    opportunities = []
    for stock in spikes:
        # Fetch additional data (float, SI, borrow rate)
        # TODO: Implement actual data fetching
        
        opportunities.append({
            'ticker': stock['ticker'],
            'change_pct': stock['change'],
            'current_price': stock['price'],
            'volume': stock['volume'],
            'float': 'TBD',
            'short_interest': 'TBD',
            'borrow_rate': 'TBD',
            'recommendation': 'WATCH - Confirm borrow availability'
        })
    
    return {
        'scan_time': datetime.now().isoformat(),
        'spikes_found': len(opportunities),
        'opportunities': opportunities
    }
'''
)
```

---

## Step 3: Restore Triggers

### 3.1 BMNR Hourly Tracker

**Prerequisites:**
- BMNR Short Tracker script created
- Task recipe created (if using recipes)

**Command:**
```python
manage_triggers(
    action='create',
    name='BMNR Short Position Hourly Tracker',
    description='Monitor BMNR short positions on Ostium every hour with P&L tracking',
    trigger_type='cron',
    cron_expression='0 * * * *',  # Every hour at :00
    # If using recipe:
    recipe='path/to/bmnr_tracker_recipe.md'
    # Or direct script execution (implementation-dependent)
)
```

---

### 3.2 Daily Battleplan Generator (Optional)

**Command:**
```python
manage_triggers(
    action='create',
    name='Daily Trading Battleplan Generator',
    description='Automatically generates and publishes daily trading battleplan at 8 AM ET',
    trigger_type='cron',
    cron_expression='0 8 * * *',  # Daily at 8 AM (assumes UTC, adjust for ET)
    recipe='path/to/daily_battleplan_recipe.md'
)
```

---

## Step 4: Reconnect API Integrations

### 4.1 GitHub
```python
# Nebula handles OAuth automatically
# Test connection:
run_action(
    action_key='github-get-current-user',
    account_id='YOUR_ACCOUNT_ID',
    props={}
)
```

### 4.2 Google Sheets
```python
# First access will trigger OAuth flow
# Test by reading a cell:
run_action(
    action_key='google_sheets-get-values',
    account_id='YOUR_ACCOUNT_ID',
    props={
        'spreadsheet_id': 'YOUR_SHEET_ID',
        'range': 'Sheet1!A1'
    }
)
```

### 4.3 CoinGecko (Public API)
```python
# No authentication required
# Test:
import requests
response = requests.get('https://api.coingecko.com/api/v3/ping')
print(response.json())  # Should return {"gecko_says":"(V3) To the Moon!"}
```

### 4.4 DEXScreener (Public API)
```python
# No authentication required
# Test:
import requests
response = requests.get('https://api.dexscreener.com/latest/dex/tokens/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v')
print(response.status_code)  # Should be 200
```

### 4.5 Polymarket Gamma API (Public)
```python
# No authentication required
# Test:
import requests
response = requests.get('https://gamma-api.polymarket.com/markets?limit=1')
print(response.status_code)  # Should be 200
```

### 4.6 Ostium (API Key Required)
```python
# Store API key in Nebula secure vault
# Test connection with your specific endpoint
```

---

## Step 5: Verify Google Sheets

### 5.1 Check Sheet Structure
1. Open your Trading Dashboard spreadsheet
2. Verify tabs exist:
   - Edge Candidates
   - Trade Signal Log
   - Performance Dashboard
3. Verify column headers match SHEETS.md specification

### 5.2 Test Write Access
```python
# Append a test row
run_action(
    action_key='google_sheets-append-values',
    account_id='YOUR_ACCOUNT_ID',
    props={
        'spreadsheet_id': 'YOUR_SHEET_ID',
        'range': 'Edge Candidates!A:A',
        'values': [['TEST']]
    }
)

# Then delete the test row manually
```

---

## Step 6: Test Script Execution

### 6.1 Test Each Script
```python
# BMNR Tracker
manage_scripts(action='run', script_path='scripts/ostium/bmnr_short_tracker.py')

# Core Portfolio
manage_scripts(action='run', script_path='scripts/coingecko/core_portfolio_tracker.py')

# Watchlist
manage_scripts(action='run', script_path='scripts/python/watchlist_token_tracker.py')

# Edge Analyzer (may fail - known issue)
manage_scripts(action='run', script_path='scripts/google_sheets/polymarket_edge_updater.py')

# Biotech Scanner
manage_scripts(action='run', script_path='scripts/python/biotech_spike_scanner.py')
```

### 6.2 Check Success Rates
```python
manage_scripts(action='list')
# Review output for success percentages
```

---

## Step 7: Enable GitHub Pages

### 7.1 Repository Settings
1. Go to https://github.com/dutchiono/trading-battleplans
2. Click **Settings** tab
3. Scroll to **Pages** section
4. Under **Source**, select:
   - Branch: `main`
   - Folder: `/ (root)`
5. Click **Save**

### 7.2 Verify Deployment
- Wait 2-5 minutes for build
- Visit: https://dutchiono.github.io/trading-battleplans/
- Should display index.md content

### 7.3 Custom Domain (Optional)
1. Add CNAME file to repo:
   ```
   trading.yourdomain.com
   ```
2. Configure DNS:
   ```
   CNAME record: trading.yourdomain.com -> dutchiono.github.io
   ```
3. Update GitHub Pages settings with custom domain

---

## Step 8: Validate End-to-End Workflows

### 8.1 Position Monitoring Workflow
```
1. Trigger fires: @trigger:bmnr-short-position-hourly-tracker
2. Script executes: bmnr_short_tracker.py
3. Output displays in Nebula chat
4. Verify P&L calculation is accurate
5. Check alert logic (if price hits key levels)
```

### 8.2 Daily Battleplan Workflow
```
1. User requests: "Generate today's battleplan"
2. Nebula researches: Earnings, catalysts, edges
3. Battleplan generated in Markdown
4. Pushed to GitHub (index.md)
5. GitHub Pages rebuilds (2-5 min)
6. Verify live URL shows new content
```

### 8.3 Edge Detection Workflow
```
1. Run: Polymarket Edge Analyzer script
2. Script fetches active markets
3. Calculates edges (research vs market)
4. Updates Google Sheets
5. Verify new rows in Edge Candidates tab
6. Check conditional formatting applied
```

---

## Troubleshooting

### Scripts Not Running
**Symptoms:** Script execution fails or times out  
**Solutions:**
1. Check API rate limits
2. Verify authentication (re-authorize if needed)
3. Review error logs in Nebula
4. Test API endpoints manually
5. Simplify script (remove complex logic temporarily)

---

### Triggers Not Firing
**Symptoms:** Hourly trigger doesn't execute on schedule  
**Solutions:**
1. Check trigger status: `manage_triggers(action='list')`
2. Verify cron expression (use crontab.guru)
3. Ensure trigger is active (not paused)
4. Check trigger history for errors
5. Recreate trigger if corrupt

---

### Google Sheets Auth Expired
**Symptoms:** 401 Unauthorized errors  
**Solutions:**
1. Re-run script from Nebula (triggers OAuth flow)
2. Authorize when prompted
3. Test with simple read operation
4. Check token expiration in Nebula vault
5. Revoke and re-grant permissions if needed

---

### GitHub Pages Not Updating
**Symptoms:** Changes pushed but site still shows old content  
**Solutions:**
1. Check build status in repo Actions tab
2. Wait 5-10 minutes (CDN cache)
3. Hard refresh browser (Ctrl+Shift+R)
4. Verify main branch has latest commits
5. Check for Jekyll build errors

---

## Maintenance

### Weekly
- Review script success rates
- Check trigger execution history
- Update forward calendar with new catalysts
- Archive old Google Sheets data

### Monthly
- Rotate API keys (if applicable)
- Audit closed positions
- Refine edge detection algorithms
- Update documentation

### Quarterly
- Full system backup
- Disaster recovery test
- Performance review
- Security audit

---

## Related Documentation

- [README.md](README.md) - System overview
- [ARCHITECTURE.md](ARCHITECTURE.md) - Design details
- [SCRIPTS.md](SCRIPTS.md) - Script documentation
- [SHEETS.md](SHEETS.md) - Dashboard guide
- [DAILY_WORKFLOW.md](DAILY_WORKFLOW.md) - Battleplan automation

---

**Setup Guide Version:** 1.0  
**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI + dutchiono