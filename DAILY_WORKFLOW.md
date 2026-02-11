# Daily Battleplan Automation Workflow

## Overview
Automated system that generates daily trading battleplans at 8 AM ET with:
- Today's intraday catalysts (earnings, FDA decisions, economic data)
- Post-spike short opportunities (biotech +40% moves)
- Forward calendar (upcoming multi-day catalysts)
- Market analysis and setup ideas
- Risk management guidelines

**IMPORTANT:** The public battleplan contains ONLY market analysis and setup ideas. Your actual positions, entry prices, and P&L are tracked privately in Nebula and NEVER published to GitHub.

---

## Architecture

### 1. Data Sources

#### PUBLIC Market Intelligence (Published to GitHub)
- **Catalyst Calendar:** Benzinga Calendar API, Earnings Whispers, FDA calendar
- **Spike Scanner:** Finviz screener for +40% biotech moves
- **Market Data:** Yahoo Finance, TradingView (for chart links)
- **News:** Benzinga News API, Reuters, web search
- **Sentiment:** Social media trends, retail interest indicators

#### PRIVATE Position Tracking (Nebula Only - NEVER Published)
- **Position Tracking:** Ostium API (BMNR short), Polymarket API
- **Portfolio Monitoring:** CoinGecko (crypto holdings)
- **Personal P&L:** Calculated from private position data
- **Trade Journal:** Your actual entries, exits, and performance

---

## Daily Workflow (8:00 AM ET Trigger)

### Phase 1: Gather PUBLIC Market Intelligence

```
Step 1: Pull Today's Catalysts
  ├─ Earnings (pre-market + intraday)
  ├─ FDA decisions/trial results
  ├─ Economic data releases (during market hours)
  └─ Analyst presentations/conferences

Step 2: Scan for Post-Spike Short Opportunities
  ├─ Run Biotech Spike Scanner
  ├─ Filter: +40% moves from yesterday
  ├─ Research: float, SI, borrow availability, catalyst
  ├─ Analyze: "priced in" risk, crowd sentiment
  └─ Calculate: generic entry/target/stop ideas (NOT your actual trades)

Step 3: Load Forward Calendar
  ├─ Upcoming catalysts (3-30 days out)
  ├─ Filter for relevance to today's market
  └─ Create "watch list" section
```

### Phase 2: Generate PUBLIC Battleplan

```
Step 4: Create Educational Content (index.md)
  ├─ Format opportunities as setup ideas (not personal trades)
  ├─ Include:
  │   ├─ Catalyst description and timing
  │   ├─ Technical levels (support/resistance)
  │   ├─ Risk assessment and "priced in" analysis
  │   ├─ Generic entry/exit ideas
  │   ├─ Crowd sentiment indicators
  │   └─ Relevant links (charts, news, filings)
  │
  ├─ Use advisory language:
  │   ✅ "Watch for entries around..."
  │   ✅ "Consider scaling in if..."
  │   ✅ "This setup offers..."
  │   ✅ "Risk/reward suggests..."
  │   ❌ "I entered at..."
  │   ❌ "My position is..."
  │   ❌ "My P&L shows..."
  │
  └─ Add risk management section (generic guidelines)
```

### Phase 3: Update PRIVATE Position Tracker

```
Step 5: Track Your Actual Trades (Nebula Only)
  ├─ Run BMNR Short Tracker (Ostium API)
  ├─ Run Core Portfolio Tracker (CoinGecko)
  ├─ Update other position monitors
  ├─ Calculate current P&L on all positions
  ├─ Update data/private_positions.json
  ├─ Add trade journal notes
  └─ NEVER commit these files to GitHub
```

**Private Position Data Structure:**
```json
{
  "last_updated": "2026-02-11T08:00:00Z",
  "positions": [
    {
      "ticker": "BMNR",
      "type": "short",
      "entry_price": 20.12,
      "current_price": 19.95,
      "size": 150,
      "unrealized_pnl_pct": 2.44,
      "stop_loss": 21.50,
      "target": 18.50,
      "notes": "Post-spike short from Feb 8 pump"
    }
  ]
}
```

### Phase 4: Publish PUBLIC Battleplan

```
Step 6: Deploy to GitHub Pages
  ├─ Archive yesterday's index.md to archive/YYYY-MM-DD.md
  ├─ Update index.md with today's PUBLIC battleplan
  ├─ Verify no private data in commit
  ├─ Commit and push to main branch
  └─ Wait for GitHub Pages build (2-5 min)
```

### Phase 5: Notify User

```
Step 7: Summary in Nebula
  ├─ Public battleplan summary (what was published)
  ├─ Private position update (P&L, action items)
  ├─ Link to live GitHub Pages
  └─ Alerts for high-priority trades
```

---

## Detailed Step Breakdown

### Step 1: Pull Today's Catalysts

**Objective:** Identify intraday opportunities based on scheduled events

**Implementation:**
```python
def fetch_todays_catalysts():
    catalysts = []
    
    # Earnings (pre-market and intraday)
    earnings = web_search(
        query="earnings today intraday stocks",
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

**Key Data Points:**
- Event type (earnings, FDA, economic)
- Time (pre-market, intraday, after-hours)
- Expected volatility/impact
- Relevant tickers

---

### Step 2: Scan for Post-Spike Shorts

**Objective:** Identify biotech stocks that spiked +40% yesterday for potential shorts today

**Implementation:**
```python
def run_biotech_spike_scanner():
    # Run existing script
    results = manage_scripts(
        action='run',
        script_path='scripts/python/biotech_spike_scanner.py'
    )
    
    # Filter for actionable opportunities
    actionable = [
        play for play in results 
        if play.get('action') != 'skip'
    ]
    
    # For each opportunity, gather:
    for play in actionable:
        # Technical analysis (chart patterns)
        play['technical'] = analyze_technicals(play['ticker'])
        
        # Sentiment analysis (Reddit, Twitter, StockTwits)
        play['sentiment'] = check_crowd_sentiment(play['ticker'])
        
        # "Priced in" risk assessment
        play['risk_score'] = calculate_priced_in_risk(play)
        
        # Generic entry/exit ideas (NOT your actual trades)
        play['setup'] = {
            'entry_zone': f"${play['support']}-${play['resistance']}",
            'target': f"${play['downside_target']}",
            'stop': f"${play['upside_stop']}",
            'position_size': '25% max (high risk)',
            'timing': 'Watch for weakness in first 30 min'
        }
    
    return actionable
```

**Output Format (PUBLIC):**
```markdown
## New Opportunity: [TICKER] ([Company Name]) ⚠️ RISK LEVEL

### The Setup
**Catalyst:** [What happened]
- Key data points
- Trial results / news summary

### Price Action
- Yesterday's move: +X%
- Current range: $X.XX - $X.XX
- Volume analysis

### "Priced In" Risk Assessment
**🔴/🟡/🟢 [RISK LEVEL] - X% Already Priced In**

Reasoning:
1. ✅/❌ Factor 1
2. ✅/❌ Factor 2
...

### Setup Ideas (Educational)
**IF conditions met:** [Market conditions]

Entry zone: $X.XX-$X.XX (generic technical levels)
Position size: X% max
Stop: $X.XX
Targets: 
- T1: $X.XX (+X%)
- T2: $X.XX (+X%)

**Avoid if:** [Red flags]
```

---

### Step 3: Load Forward Calendar

**Objective:** Surface upcoming catalysts worth monitoring

**Implementation:**
```python
def load_forward_calendar():
    # Load from data/forward_calendar.json
    with open('data/forward_calendar.json', 'r') as f:
        calendar = json.load(f)
    
    # Filter for relevant upcoming events
    today = datetime.now()
    relevant = [
        event for event in calendar['events']
        if today <= event['date'] <= today + timedelta(days=30)
    ]
    
    # Group by week
    by_week = {
        'this_week': [],
        'next_week': [],
        'beyond': []
    }
    
    for event in relevant:
        days_out = (event['date'] - today).days
        if days_out <= 7:
            by_week['this_week'].append(event)
        elif days_out <= 14:
            by_week['next_week'].append(event)
        else:
            by_week['beyond'].append(event)
    
    return by_week
```

---

### Step 4: Generate PUBLIC Battleplan

**Objective:** Create educational markdown content with clear separation from personal trades

**Template Structure:**
```markdown
# [Day], [Date] - Intraday Trading Battleplan

## Market Overview
[Brief market context, key themes for the day]

---

## Today's Catalyst Calendar
- **Pre-Market:** [List]
- **Intraday:** [List]
- **After-Hours:** [List]

---

## Setup #1: [Opportunity Name]

[Full setup details as shown in Step 2 format]

---

## Setup #2: [Another Opportunity]

[...]

---

## Forward Calendar: Upcoming Catalysts

### This Week ([Date Range])
- [Event 1]
- [Event 2]

### Next Week ([Date Range])
- [Event 1]

### Beyond ([Date Range])
- [Event 1]

---

## Risk Management Guidelines
- Max position size: X% per trade
- Stop losses mandatory
- Take profits at targets
- Daily loss limit: X%

---

## Key Links
- [Chart links]
- [News sources]
- [Calendar links]

---

**Last Updated:** [Timestamp]
**Next Update:** [Tomorrow's date] at 8:00 AM EST (automated)
```

**Critical Rules:**
- ❌ NO "Current Position" sections
- ❌ NO personal entry prices or P&L
- ❌ NO "My trade" language
- ✅ Use "Consider...", "Watch for...", "Setup available..."
- ✅ Present as educational/advisory content
- ✅ Generic technical levels, not your actual stops

---

### Step 5: Update PRIVATE Positions

**Objective:** Track your actual trades separately from public battleplan

**Implementation:**
```python
def update_private_positions():
    positions = {}
    
    # BMNR short position (if active)
    bmnr = manage_scripts(
        action='run',
        script_path='scripts/ostium/bmnr_short_tracker.py'
    )
    if bmnr['has_position']:
        positions['BMNR'] = {
            'type': 'short',
            'entry': bmnr['entry_price'],
            'current': bmnr['current_price'],
            'pnl_pct': bmnr['pnl_pct'],
            'liquidation': bmnr['liquidation_price'],
            'action': bmnr['recommendation']
        }
    
    # Crypto portfolio
    crypto = manage_scripts(
        action='run',
        script_path='scripts/coingecko/core_portfolio_tracker.py'
    )
    positions['crypto'] = crypto['portfolio']
    
    # Polymarket positions (if any)
    # ... additional position trackers
    
    # Save to PRIVATE file (never committed to Git)
    with open('data/private_positions.json', 'w') as f:
        json.dump({
            'last_updated': datetime.now().isoformat(),
            'positions': positions
        }, f, indent=2)
    
    return positions
```

**Private Position Display (Nebula Chat Only):**
```
📊 Your Private Position Update:

BMNR Short:
  Entry: $20.12
  Current: $19.95 (+2.44% P&L)
  Liquidation: $27.00
  Action: HOLD or EXIT at open based on overnight action

Crypto Portfolio:
  BTC: $XX,XXX (+X.X%)
  ETH: $X,XXX (+X.X%)
  Total: $XX,XXX (+X.X%)
```

---

### Step 6: Publish to GitHub

**Objective:** Deploy public battleplan while keeping private data safe

**Implementation:**
```python
def publish_to_github():
    # 1. Archive yesterday's battleplan
    yesterday = (datetime.now() - timedelta(days=1)).strftime('%Y-%m-%d')
    
    # Get current index.md
    current = run_action(
        action_key='github-get-repository-content',
        props={'repoFullname': 'dutchiono/trading-battleplans', 'path': 'index.md'},
        account_id='apn_XehbgoX'
    )
    
    # Archive it
    run_action(
        action_key='github-create-or-update-file-contents',
        props={
            'repoFullname': 'dutchiono/trading-battleplans',
            'path': f'archive/{yesterday}.md',
            'fileContent': current['content'],
            'commitMessage': f'Archive battleplan from {yesterday}'
        },
        account_id='apn_XehbgoX'
    )
    
    # 2. Generate today's PUBLIC battleplan
    battleplan = generate_public_battleplan()
    
    # 3. VERIFY no private data before committing
    if has_private_data(battleplan):
        raise Exception("ERROR: Private data detected in battleplan! Aborting publish.")
    
    # 4. Update index.md
    run_action(
        action_key='github-create-or-update-file-contents',
        props={
            'repoFullname': 'dutchiono/trading-battleplans',
            'path': 'index.md',
            'fileContent': battleplan,
            'commitMessage': f'Update battleplan for {datetime.now().strftime("%Y-%m-%d")}'
        },
        account_id='apn_XehbgoX'
    )
    
    return True
```

**Safety Checks:**
```python
def has_private_data(content):
    """Check for accidental private data leaks"""
    red_flags = [
        'Entry: $',
        'Current: $',
        'P&L',
        'My position',
        'I entered',
        'liquidation: $',
        'unrealized',
        'position size: X shares'
    ]
    
    for flag in red_flags:
        if flag.lower() in content.lower():
            return True
    
    return False
```

---

## Monitoring & Alerts

### Success Indicators
- ✅ Battleplan published by 8:05 AM ET
- ✅ GitHub Pages deployed within 5 minutes
- ✅ No private data in public repository
- ✅ Private positions updated in Nebula

### Failure Recovery
If automation fails:
1. Check trigger status: `manage_triggers(action='get', trigger_slug='daily-trading-battleplan-generator')`
2. Review error logs in Nebula chat
3. Manual fallback: Run task recipe manually via `manage_tasks(action='load_recipe', recipe='...')`
4. Emergency: Generate battleplan manually and push via GitHub UI

---

## File Organization

### GitHub Repository (PUBLIC)
```
dutchiono/trading-battleplans/
├── index.md                    # Today's battleplan (PUBLIC)
├── archive/                    
│   ├── 2026-02-10.md           # Yesterday's battleplan
│   └── 2026-02-09.md           # Historical
├── ARCHITECTURE.md             # System documentation
├── DAILY_WORKFLOW.md           # This file
├── SETUP.md                    # Setup instructions
└── data/
    └── forward_calendar.json   # Upcoming catalysts (PUBLIC)
```

### Nebula Workspace (PRIVATE - Gitignored)
```
data/
├── private_positions.json      # Your actual trades
├── position_history.json       # Historical P&L
├── trade_journal.json          # Trade notes
└── forward_calendar.json       # LOCAL copy for updates
```

### `.gitignore` Configuration
```
# Private position data - NEVER commit
data/private_*.json
data/position_*.json
data/trade_*.json

# Local environment
.env
.env.local

# Python
__pycache__/
*.pyc
```

---

## Best Practices

### Public Content (GitHub)
- ✅ Educational and advisory tone
- ✅ Generic setup ideas with technical levels
- ✅ Risk analysis and "priced in" assessments
- ✅ Market research and catalyst information
- ❌ Personal positions, entries, or P&L
- ❌ "I" or "my" when discussing trades
- ❌ Actual stop loss levels of active positions

### Private Tracking (Nebula)
- ✅ Detailed position data with exact entries
- ✅ Real-time P&L calculations
- ✅ Personal trade journal and notes
- ✅ Strategy performance metrics
- ❌ Never commit to GitHub
- ❌ Don't mix with public content

---

**Last Updated:** February 11, 2026  
**Next Review:** When workflow processes change
