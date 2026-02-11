# Daily Battleplan Automation Workflow

## Overview
Automated system that generates daily trading battleplans at 8 AM ET with:
- Today's intraday catalysts (earnings, FDA decisions, economic data)
- Post-spike short opportunities (biotech +40% moves)
- Forward calendar (upcoming multi-day catalysts)
- Your existing positions with updated P&L
- Relevant links (charts, news, filings)

## Architecture

### 1. Data Sources
- **Catalyst Calendar:** Benzinga Calendar API, Earnings Whispers, FDA calendar
- **Spike Scanner:** Finviz screener for +40% biotech moves
- **Position Tracking:** Ostium API (BMNR short), Polymarket API
- **Market Data:** Yahoo Finance, TradingView
- **News:** Benzinga News API, Reuters

### 2. Daily Workflow (8:00 AM ET Trigger)

```
Step 1: Pull Today's Catalysts
  ├─ Earnings (pre-market + intraday)
  ├─ FDA decisions/trial results
  ├─ Economic data releases (during market hours)
  └─ Analyst presentations/conferences

Step 2: Scan for Post-Spike Shorts
  ├─ Run Biotech Spike Scanner
  ├─ Filter: +40% moves from yesterday
  ├─ Check float, SI, borrow availability
  └─ Calculate entry/target/stop levels

Step 3: Update Existing Positions
  ├─ BMNR short position (Ostium API)
  ├─ Polymarket positions (if any)
  ├─ Crypto positions (Core Portfolio script)
  └─ Calculate P&L and risk levels

Step 4: Check Forward Calendar
  ├─ Load upcoming catalysts (3-30 days out)
  ├─ Filter for relevance to today
  └─ Add "watch list" section

Step 5: Generate Battleplan
  ├─ Rank opportunities by risk/reward
  ├─ Add relevant links for each play
  ├─ Include technical charts
  ├─ Set alerts and timing strategy
  └─ Format as markdown with sections

Step 6: Publish to GitHub Pages
  ├─ Update index.md with today's plan
  ├─ Archive yesterday's plan to /archive
  ├─ Commit and push to main branch
  └─ Wait for GitHub Pages build (2-5 min)

Step 7: Notify User
  ├─ Post summary in Nebula chat
  ├─ Send email/Telegram alert (optional)
  └─ Include link to live battleplan
```

---

## Detailed Step Breakdown

### Step 1: Pull Today's Catalysts

**Objective:** Identify intraday opportunities based on scheduled events

**Implementation:**
```python
def fetch_todays_catalysts():
    catalysts = []
    
    # Earnings (pre-market)
    earnings = web_search(
        query="earnings today pre-market stocks",
        category="news",
        num_results=5
    )
    
    # FDA calendar
    fda = web_search(
        query="FDA decision calendar today",
        category="news",
        num_results=3
    )
    
    # Economic data
    econ = web_search(
        query="economic data releases today schedule",
        category="news",
        num_results=3
    )
    
    # Parse and structure
    for item in [earnings, fda, econ]:
        catalysts.append(parse_catalyst(item))
    
    return catalysts
```

**Data Structure:**
```json
{
  "date": "2026-02-11",
  "catalysts": [
    {
      "type": "earnings",
      "ticker": "TMUS",
      "company": "T-Mobile",
      "time": "pre_market",
      "expected_move": "3-5%",
      "historical_beat_rate": 0.90,
      "links": {
        "earnings_call": "https://...",
        "estimates": "https://..."
      }
    }
  ]
}
```

---

### Step 2: Scan for Post-Spike Shorts

**Objective:** Find biotech stocks that spiked yesterday, now ready to fade

**Implementation:**
```python
def scan_biotech_spikes():
    # Run Biotech Spike Scanner script
    result = manage_scripts(
        action='run',
        script_path='scripts/python/biotech_spike_scanner.py'
    )
    
    spikes = result['opportunities']
    
    # Enrich with short data
    for spike in spikes:
        spike['borrow_rate'] = fetch_borrow_rate(spike['ticker'])
        spike['float'] = fetch_float(spike['ticker'])
        spike['short_interest'] = fetch_short_interest(spike['ticker'])
        spike['priced_in_score'] = calculate_priced_in(spike)
    
    # Filter: Only include if shortable
    return [s for s in spikes if s['borrow_rate'] and s['borrow_rate'] < 50]
```

**Research Checklist (Per Spike):**
- [ ] What was the catalyst? (FDA, earnings, partnership)
- [ ] Has news fully disseminated? (24h+ since announcement)
- [ ] Historical pattern? (Does this company have pump/dump history)
- [ ] Insider activity? (Recent Form 4 filings)
- [ ] Short squeeze risk? (SI%, borrow rate, days to cover)

---

### Step 3: Update Existing Positions

**Objective:** Show current P&L and exit recommendations

**Implementation:**
```python
def update_positions():
    positions = []
    
    # BMNR Short
    bmnr = manage_scripts(
        action='run',
        script_path='scripts/ostium/bmnr_short_tracker.py'
    )
    positions.append(format_bmnr_position(bmnr))
    
    # Polymarket positions (if any)
    pm_positions = fetch_polymarket_positions()
    positions.extend([format_pm_position(p) for p in pm_positions])
    
    # Crypto holdings
    crypto = manage_scripts(
        action='run',
        script_path='scripts/coingecko/core_portfolio_tracker.py'
    )
    positions.append(format_crypto_holdings(crypto))
    
    return positions
```

**Output Format:**
```markdown
## Current Positions

### BMNR Short (Ostium 3x)
- Entry: $20.12 | Current: $19.95 | P&L: +2.44%
- Liquidation: $27.00 (34.2% away)
- Status: HOLD - Target $19.50 for profit exit
- [Live Chart](https://www.tradingview.com/chart/?symbol=BMNR)

### Crypto Holdings
- BTC: $98,234 (+2.3% 24h)
- ETH: $3,456 (+1.8% 24h)
- SOL: $145 (+5.2% 24h)
```

---

### Step 4: Check Forward Calendar

**Objective:** Surface upcoming catalysts that need preparation

**Implementation:**
```python
def load_forward_calendar():
    # Load from data/forward_calendar.json
    with open('data/forward_calendar.json', 'r') as f:
        calendar = json.load(f)
    
    today = datetime.now().date()
    relevant = []
    
    for catalyst in calendar['catalysts']:
        cat_date = datetime.strptime(catalyst['date'], '%Y-%m-%d').date()
        days_away = (cat_date - today).days
        
        # Include if 3-30 days out
        if 3 <= days_away <= 30:
            catalyst['days_away'] = days_away
            relevant.append(catalyst)
    
    return sorted(relevant, key=lambda x: x['days_away'])
```

**Forward Calendar Structure:**
```json
{
  "updated_at": "2026-02-11T01:59:00Z",
  "catalysts": [
    {
      "date": "2026-02-15",
      "ticker": "NVDA",
      "company": "NVIDIA",
      "catalyst_type": "earnings",
      "expected_move": "5-8%",
      "notes": "Q4 results - AI data center revenue key metric"
    }
  ]
}
```

---

### Step 5: Generate Battleplan

**Objective:** Synthesize all data into actionable markdown

**Template:**
```markdown
# [Day], [Date] - Intraday Trading Battleplan

## Current Positions
[From Step 3]

## New Opportunity #1: [Ticker]
### The Setup
- Catalyst: [What happened]
- Price Action: [How it moved]
- Crowd Sentiment: [Retail hype level]

### Technical Levels
- Entry: [Price range]
- Target: [Exit price]
- Stop: [Risk limit]

### Priced In Risk Assessment
[Score with reasoning]

### Trade Checklist
- [ ] Verify borrow availability
- [ ] Check float and SI%
- [ ] Set alerts at key levels

---

## Forward Calendar

### This Week
- [Date]: [Ticker] - [Catalyst]

### Next 2-4 Weeks
- [Date]: [Ticker] - [Catalyst]

---

## Today's Gameplan

### Pre-Market (7:00-9:30 AM)
1. [Action item]

### Market Hours (9:30 AM - 4:00 PM)
1. [Action item]

### After Hours (4:00-8:00 PM)
1. [Action item]

---

## Risk Management
- Max loss per trade: [Amount]
- Max portfolio heat: [Percentage]
- Position sizing: [Formula]

---

## Notes
[Any additional context]
```

---

### Step 6: Publish to GitHub Pages

**Objective:** Make battleplan publicly accessible

**Implementation:**
```python
def publish_battleplan(markdown_content):
    # 1. Archive yesterday's plan
    yesterday = (datetime.now() - timedelta(days=1)).strftime('%Y-%m-%d')
    archive_path = f'archive/{yesterday}_battleplan.md'
    
    # Move current index.md to archive (if exists)
    run_action(
        action_key='github-get-repository-content',
        account_id='apn_XehbgoX',
        props={'repoFullname': 'dutchiono/trading-battleplans', 'path': 'index.md'}
    )
    # ... (archive logic)
    
    # 2. Update index.md with today's plan
    run_action(
        action_key='github-create-or-update-file-contents',
        account_id='apn_XehbgoX',
        props={
            'repoFullname': 'dutchiono/trading-battleplans',
            'path': 'index.md',
            'fileContent': markdown_content,
            'commitMessage': f'Update battleplan for {datetime.now().strftime("%Y-%m-%d")}',
            'branch': 'main'
        }
    )
    
    # 3. Wait for GitHub Pages build
    time.sleep(180)  # 3 minutes
    
    # 4. Return live URL
    return 'https://dutchiono.github.io/trading-battleplans/'
```

---

### Step 7: Notify User

**Objective:** Alert user that battleplan is ready

**Implementation:**
```python
def notify_user(battleplan_url, summary):
    message = f"""
🎯 Daily Battleplan Generated!

📊 Summary:
- Current Positions: {summary['positions_count']}
- New Opportunities: {summary['opportunities_count']}
- Forward Catalysts: {summary['calendar_count']}

🔗 Live Battleplan: {battleplan_url}

✅ Next Steps:
1. Review BMNR position status
2. Check borrow availability for new shorts
3. Set alerts for key price levels
    """
    
    # Post in Nebula chat (automatic)
    print(message)
    
    # Optional: Send email
    # send_email(to='dutchiono@gmail.com', subject='Daily Battleplan', body=message)
    
    # Optional: Send Telegram
    # send_telegram(chat_id='YOUR_CHAT_ID', text=message)
```

---

## Task Recipe (For Trigger)

**File:** `recipes/daily_battleplan_generator/TASK.md`

```markdown
---
title: Daily Trading Battleplan Generator
description: Generates and publishes daily trading battleplan at 8 AM ET
---

## Steps

1. Fetch today's catalysts (earnings, FDA, economic data)
2. Run biotech spike scanner for post-catalyst shorts
3. Update existing positions (BMNR, crypto, Polymarket)
4. Load forward calendar (3-30 days out)
5. Generate markdown battleplan
6. Archive yesterday's plan to /archive
7. Publish to index.md on GitHub
8. Notify user with summary

## Expected Output
- Updated index.md with today's battleplan
- Archived previous day's battleplan
- Summary posted in Nebula chat
- Live URL: https://dutchiono.github.io/trading-battleplans/
```

---

## Trigger Configuration

```python
manage_triggers(
    action='create',
    name='Daily Trading Battleplan Generator',
    description='Automatically generates and publishes daily trading battleplan at 8 AM ET with catalysts, opportunities, and forward calendar',
    trigger_type='cron',
    cron_expression='0 13 * * *',  # 8 AM ET = 1 PM UTC (DST-adjusted)
    recipe='recipes/daily_battleplan_generator/TASK.md',
    is_active=True
)
```

**Note:** Adjust cron for EST vs EDT (UTC-5 vs UTC-4)

---

## Manual Execution

```python
# Generate battleplan on-demand
"Generate today's trading battleplan"

# Or load and execute recipe
manage_tasks(
    action='load_recipe',
    recipe='recipes/daily_battleplan_generator/TASK.md'
)
```

---

## Customization Options

### Filter Criteria
```python
# Adjust biotech spike threshold
SPIKE_THRESHOLD = 40  # Default: +40%

# Adjust forward calendar window
FORWARD_DAYS_MIN = 3
FORWARD_DAYS_MAX = 30

# Adjust opportunity ranking
RANK_BY = 'risk_reward'  # Options: 'edge', 'conviction', 'risk_reward'
```

### Content Sections
```python
# Enable/disable sections
INCLUDE_POSITIONS = True
INCLUDE_OPPORTUNITIES = True
INCLUDE_FORWARD_CALENDAR = True
INCLUDE_GAMEPLAN = True
INCLUDE_RISK_MANAGEMENT = True
```

### Notification Channels
```python
# Configure alerts
NOTIFY_NEBULA = True  # Always enabled
NOTIFY_EMAIL = False  # Optional
NOTIFY_TELEGRAM = False  # Optional
```

---

## Monitoring

### Success Metrics
- ✅ Trigger fires daily at 8:00 AM ET
- ✅ Battleplan generation completes in <5 minutes
- ✅ GitHub Pages deploys within 5 minutes
- ✅ Live URL accessible and up-to-date

### Failure Scenarios

**Trigger Doesn't Fire:**
- Check trigger status (active/paused)
- Verify cron expression
- Check Nebula trigger history

**Battleplan Generation Fails:**
- Check API rate limits (web search, market data)
- Verify script execution logs
- Test individual components (catalyst fetch, spike scan, etc.)

**GitHub Push Fails:**
- Check authentication (OAuth token)
- Verify repository permissions
- Test manual push

**Pages Build Fails:**
- Check GitHub Actions tab for errors
- Verify markdown syntax (Jekyll compatibility)
- Test locally with Jekyll

---

## Related Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [SCRIPTS.md](SCRIPTS.md) - Script details
- [SETUP.md](SETUP.md) - Installation guide
- [README.md](README.md) - Quick start

---

**Workflow Version:** 1.0  
**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI + dutchiono