# System Architecture

Complete architectural overview of the Nebula trading automation system.

## Table of Contents
1. [System Overview](#system-overview)
2. [Core Components](#core-components)
3. [Data Flow](#data-flow)
4. [Integration Map](#integration-map)
5. [Automation Workflows](#automation-workflows)
6. [Security & Authentication](#security--authentication)
7. [Deployment Architecture](#deployment-architecture)

---

## System Overview

### Mission
Automate trading intelligence, position monitoring, and opportunity detection across crypto and prediction markets while maintaining a daily battleplan for strategic decision-making.

### Design Philosophy
- **Modular**: Each script is independent and reusable
- **Event-Driven**: Triggers orchestrate scheduled execution
- **Observable**: All data flows through central dashboard (Google Sheets)
- **Resilient**: Graceful failure handling and retry logic
- **Documented**: Every component has clear purpose and usage instructions

---

## Core Components

### 1. Nebula AI Agent
**Role:** Central orchestrator and conversational interface

**Capabilities:**
- Natural language task understanding
- Script creation and management
- Trigger scheduling and monitoring
- API authentication handling
- File/document management
- Delegation to specialized agents

**Technologies:**
- Pydantic AI for structured outputs
- Function calling for tool execution
- Context management for conversation history

---

### 2. Scripts (Automation Layer)

#### 2.1 BMNR Short Tracker
```
Location: scripts/ostium/bmnr_short_tracker.py
Purpose: Monitor Ostium perpetual short position
Frequency: Hourly (trigger-based)
Success Rate: 100%
```

**Data Flow:**
```
Ostium API → Script → Position Analysis → Alert/Report
```

**Key Metrics:**
- Entry price: $20.12
- Current P&L percentage
- Liquidation distance
- Exit recommendations

---

#### 2.2 Core Portfolio Tracker
```
Location: scripts/coingecko/core_portfolio_tracker.py
Purpose: Track BTC, ETH, SOL, XMR, MONAD prices
Frequency: On-demand / Hourly
Success Rate: 100%
```

**Data Flow:**
```
CoinGecko API → Script → Price Aggregation → Portfolio Summary
```

**Assets Tracked:**
- Bitcoin (BTC)
- Ethereum (ETH)
- Solana (SOL)
- Monero (XMR)
- Monad (MONAD)

---

#### 2.3 Watchlist Token Tracker
```
Location: scripts/python/watchlist_token_tracker.py
Purpose: Monitor custom tokens on Solana and Base
Frequency: On-demand
Success Rate: 100%
```

**Data Flow:**
```
DEXScreener API → Script → Price Analysis → Volatility Alerts
```

**Alert Conditions:**
- +20% in 1h (pump alert)
- -15% in 1h (dump alert)
- 3x volume spike
- Liquidity <$100K (rug risk)

---

#### 2.4 Polymarket Edge to Sheets Updater
```
Location: scripts/google_sheets/polymarket_edge_updater.py
Purpose: Identify mispriced prediction markets
Frequency: Manual / Scheduled
Success Rate: 0% (debugging needed)
```

**Data Flow:**
```
Polymarket API → Edge Detection → Google Sheets → Alerts
                ↓
        Web Research (polls, news, models)
```

**Edge Detection Algorithm:**
1. Fetch active markets from Polymarket
2. Scrape external probability sources
3. Calculate edge = Research Probability - Market Probability
4. Rank by edge size and liquidity
5. Update Google Sheets with findings

**Known Issues:**
- Google Sheets API authentication
- Rate limiting on web scraping
- Data schema validation failures

---

#### 2.5 Bi-Directional Volatility Scanner
```
Location: scripts/python/bidirectional_volatility_scanner.py
Purpose: Detect pump/dump opportunities
Frequency: Planned hourly
Success Rate: 0% (not yet implemented)
```

**Planned Data Flow:**
```
CryptoCompare + DEXScreener + CoinGecko
         ↓
Technical Analysis (RSI, Volume, Price Change)
         ↓
Signal Generation (LONG/SHORT opportunities)
         ↓
Telegram Alerts + Google Sheets Log
```

**Detection Criteria:**
- **Short Setup:** +30% in 4h, RSI >80, volume spike
- **Long Setup:** -40% from high, RSI <30, panic selling

---

### 3. Triggers (Scheduling Layer)

#### Active Triggers

**BMNR Hourly Tracker**
```yaml
Name: @trigger:bmnr-short-position-hourly-tracker
Type: cron
Schedule: 0 * * * * (every hour)
Recipe: BMNR Short Position Tracker
Status: Active
Next Run: (dynamic)
```

**Ken Paxton NO Position Tracker (Paused)**
```yaml
Name: @trigger:ken-paxton-no-position-hourly-tracker
Type: cron
Schedule: 0 * * * *
Status: Paused
Reason: Position closed or on hold
```

#### Trigger Management
```python
# List all triggers
manage_triggers(action='list')

# Get trigger details
manage_triggers(action='get', trigger_slug='bmnr-short-position-hourly-tracker')

# Update trigger
manage_triggers(action='update', trigger_slug='...', is_active=False)

# Delete trigger
manage_triggers(action='delete', trigger_slug='...')
```

---

### 4. Google Sheets (Data Dashboard)

**Purpose:** Central visualization and tracking dashboard

**Tabs:**
1. **Edge Candidates** - Polymarket opportunities
2. **Trade Signal Log** - Historical signals
3. **Performance Dashboard** - KPIs and charts

**Integration:**
- Google Sheets API (OAuth 2.0)
- Automated updates via scripts
- Manual entry for trade outcomes
- Conditional formatting for alerts

See [SHEETS.md](SHEETS.md) for complete documentation.

---

### 5. GitHub Pages (Public Documentation)

**Repository:** https://github.com/dutchiono/trading-battleplans  
**Live Site:** https://dutchiono.github.io/trading-battleplans/

**Content:**
- Daily trading battleplans (index.md)
- System documentation (SCRIPTS.md, SHEETS.md, ARCHITECTURE.md)
- Setup instructions (SETUP.md)
- README with quick links

**Update Workflow:**
1. Generate battleplan via Nebula
2. Convert to Markdown with Jekyll front matter
3. Push to main branch
4. GitHub Pages auto-deploys (2-5 min)

---

## Data Flow

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────┐
│                      Nebula AI Agent                    │
│  (Orchestrator, Conversation Interface, Task Manager)  │
└────────────┬────────────────────────┬──────────────────┘
             │                        │
             ▼                        ▼
    ┌────────────────┐      ┌────────────────┐
    │    Scripts     │      │   Triggers     │
    │  (Automation)  │◄─────┤  (Scheduler)   │
    └────────┬───────┘      └────────────────┘
             │
             ▼
    ┌────────────────────────────────────────┐
    │         External APIs                   │
    │  • Ostium (Perps)                      │
    │  • CoinGecko (Prices)                  │
    │  • DEXScreener (DEX Data)              │
    │  • Polymarket (Prediction Markets)     │
    │  • Google Sheets (Dashboard)           │
    │  • GitHub (Documentation)              │
    └────────────────────────────────────────┘
             │
             ▼
    ┌────────────────────────────────────────┐
    │         Outputs                         │
    │  • Position alerts                      │
    │  • Trading opportunities                │
    │  • Daily battleplans                    │
    │  • Google Sheets updates                │
    │  • Telegram notifications (planned)     │
    └────────────────────────────────────────┘
```

---

### Detailed Data Flow: Daily Battleplan

```
1. User Request
   "Create tomorrow's trading battleplan"
   │
   ▼
2. Nebula Agent
   - Parse request
   - Identify required data sources
   - Delegate research tasks
   │
   ▼
3. Market Research
   ├─► Web Search: Earnings calendars, news, catalysts
   ├─► CoinGecko: Crypto prices and trends
   ├─► Polymarket: Active prediction markets
   └─► DEXScreener: Token momentum
   │
   ▼
4. Analysis & Ranking
   - Identify opportunities (stocks, crypto, prediction markets)
   - Calculate "priced in" risk scores
   - Rank by risk/reward and conviction
   - Generate entry/exit levels
   │
   ▼
5. Battleplan Generation
   - Structure opportunities (1, 2, 3...)
   - Add risk management rules
   - Include checklist and gameplan
   - Format as Markdown
   │
   ▼
6. GitHub Pages Deployment
   - Convert to Jekyll format (index.md)
   - Push to trading-battleplans repo
   - Auto-deploy via GitHub Pages
   │
   ▼
7. Access
   User views: https://dutchiono.github.io/trading-battleplans/
```

---

### Detailed Data Flow: Position Monitoring

```
1. Cron Trigger Fires
   @trigger:bmnr-short-position-hourly-tracker
   │
   ▼
2. Script Execution
   scripts/ostium/bmnr_short_tracker.py
   - Fetch current BMNR price
   - Load position details (entry: $20.12)
   - Calculate P&L percentage
   │
   ▼
3. Risk Assessment
   - Distance to liquidation ($27.00)
   - Compare to alert levels ($19.00, $19.50, $20.50)
   - Generate recommendation (HOLD / EXIT / STOP)
   │
   ▼
4. Output
   ├─► Nebula Chat: Position summary
   ├─► Telegram (planned): Alert if critical level
   └─► Log: Store execution result
```

---

### Detailed Data Flow: Edge Detection

```
1. Manual Trigger or Schedule
   Run Polymarket Edge Detector script
   │
   ▼
2. Market Discovery
   Polymarket Gamma API → Fetch active markets
   - Filter by category (Sports, Politics, Crypto)
   - Filter by liquidity (>$10K)
   - Filter by time to close (>7 days preferred)
   │
   ▼
3. External Research (Per Market)
   ├─► Web Search: Polls, expert predictions, news
   ├─► Web Scrape: 538, RealClearPolitics, betting sites
   └─► Analysis: Weighted probability estimate
   │
   ▼
4. Edge Calculation
   Edge = Research Probability - Market Implied Probability
   - Positive edge = Market undervalues outcome
   - Negative edge = Market overvalues outcome
   │
   ▼
5. Risk Scoring
   Composite risk score (1-10):
   - Low liquidity = +3
   - Low volume = +2
   - Short time to close = +2
   - No research = +5
   - Huge edge = -2 (reduces risk)
   │
   ▼
6. Google Sheets Update
   - Append new rows to Edge Candidates tab
   - Update existing markets (price refresh)
   - Sort by edge percentage
   - Apply conditional formatting
   │
   ▼
7. Alerts
   If edge >15% AND risk <5:
   - High-conviction opportunity
   - Send Telegram notification (planned)
   - Post to Nebula chat
```

---

## Integration Map

### API Integrations

| Service | Purpose | Authentication | Rate Limits | Documentation |
|---------|---------|----------------|-------------|---------------|
| **Ostium** | Perpetual futures positions | API Key | TBD | Internal |
| **CoinGecko** | Crypto prices, market data | Free tier (no key) | 10-50 req/min | [Link](https://www.coingecko.com/api/documentation) |
| **DEXScreener** | DEX pair data | Public API | Generous | [Link](https://docs.dexscreener.com/) |
| **Polymarket Gamma** | Prediction markets | Public API | 100 req/min | [Link](https://docs.polymarket.com/) |
| **Google Sheets** | Dashboard updates | OAuth 2.0 | 100 req/100s | [Link](https://developers.google.com/sheets/api) |
| **GitHub** | Documentation hosting | OAuth (Nebula) | 5000 req/hr | [Link](https://docs.github.com/rest) |
| **CryptoCompare** | OHLCV data (planned) | API Key | 100K req/month | [Link](https://min-api.cryptocompare.com/) |

---

### Data Storage

| Layer | Technology | Purpose | Persistence |
|-------|------------|---------|-------------|
| **Conversation** | Nebula Memory | Chat history, context | Persistent |
| **Scripts** | Nebula Scripts | Automation code | Persistent |
| **Triggers** | Nebula Triggers | Scheduled jobs | Persistent |
| **Dashboard** | Google Sheets | Position tracking, analysis | Persistent |
| **Documentation** | GitHub Repo | Public docs, battleplans | Persistent (Git) |
| **Execution Logs** | Nebula Logs | Script run history | 30-day retention |
| **Temporary Files** | Nebula tmp/ | Intermediate outputs | Session-scoped |

---

## Automation Workflows

### Workflow 1: Hourly Position Check

```yaml
Name: BMNR Position Monitoring
Trigger: @trigger:bmnr-short-position-hourly-tracker
Schedule: Every hour at :00

Steps:
  1. Trigger fires at top of hour
  2. Load task recipe: BMNR Short Position Tracker
  3. Execute script: bmnr_short_tracker.py
     - Fetch current price from Ostium
     - Calculate P&L vs entry ($20.12)
     - Assess liquidation risk ($27.00)
     - Generate recommendation
  4. Output to Nebula chat
  5. (Future) Send Telegram alert if critical

Success Criteria:
  - Script executes without errors
  - Price data is current (<5 min old)
  - P&L calculation is accurate
  - Recommendation matches predefined logic

Failure Handling:
  - Retry once after 5 min
  - If still fails, alert user via Nebula
  - Log error for debugging
```

---

### Workflow 2: Daily Battleplan Generation

```yaml
Name: Daily Trading Battleplan
Trigger: Manual or scheduled (planned)
Frequency: Daily at 6:00 AM EST (before market open)

Steps:
  1. User requests: "Generate today's battleplan"
  2. Nebula researches:
     - Earnings calendar (TMUS, CSCO, etc.)
     - Crypto momentum (parabolic moves, dumps)
     - News catalysts (FDA approvals, geopolitics)
     - Prediction market edges (Polymarket)
  3. Web scraping:
     - Earnings reports and consensus estimates
     - Historical beat/miss rates
     - Analyst recommendations
     - Reddit/Twitter sentiment (optional)
  4. Analysis:
     - Rank opportunities by risk/reward
     - Calculate "priced in" percentages
     - Define entry/exit/stop levels
     - Assign conviction (Low/Medium/High)
  5. Document generation:
     - Markdown with Jekyll front matter
     - Sections: Current positions, New opportunities, Gameplan
     - Formatting: Risk badges, checklists, tables
  6. GitHub deployment:
     - Push index.md to trading-battleplans repo
     - Wait for Pages build (2-5 min)
     - Confirm live URL
  7. Notify user: Link to battleplan

Success Criteria:
  - All research sources queried successfully
  - At least 3 opportunities identified
  - Risk scores calculated for each
  - Battleplan published to GitHub Pages
  - Accessible at public URL

Failure Handling:
  - If research fails: Use cached data + disclaimer
  - If GitHub push fails: Save locally, manual push
  - If no opportunities: Publish "sit in cash" battleplan
```

---

### Workflow 3: Edge Detection & Sheets Update

```yaml
Name: Polymarket Edge Detection
Trigger: Manual or scheduled (planned hourly)
Frequency: Every 4 hours (recommended)

Steps:
  1. Trigger fires or user requests
  2. Execute script: polymarket_edge_updater.py
  3. Fetch markets:
     - Query Polymarket Gamma API
     - Filter: Active, >$10K liquidity, resolves in >7 days
     - Target categories: Sports, Politics, Crypto
  4. Research each market:
     - Web search: Polls, betting sites, news
     - Scrape: Specific sources (538, PredictIt, etc.)
     - Analyze: Calculate research-based probability
  5. Calculate edge:
     - Edge = Research Probability - Market Probability
     - Filter: Only edges >5% (meaningful)
  6. Risk scoring:
     - Composite score based on liquidity, volume, time
     - 1-10 scale (1=low risk, 10=extreme)
  7. Google Sheets update:
     - Authenticate via OAuth
     - Check for duplicates (market ID)
     - Append new rows or update existing
     - Sort by edge (descending)
  8. Alerts:
     - If edge >15% AND risk <5: High-conviction alert
     - Post summary to Nebula chat
     - (Future) Send Telegram notification

Success Criteria:
  - At least 5 markets analyzed
  - Edge calculations accurate
  - Google Sheets updated without errors
  - No duplicate entries
  - Conditional formatting applied

Failure Handling:
  - If API fails: Retry with exponential backoff
  - If Sheets auth fails: Trigger re-authentication
  - If scraping fails: Use cached data, lower conviction
  - If no edges found: Log "no opportunities" message
```

---

## Security & Authentication

### Authentication Methods

| Service | Method | Storage | Renewal |
|---------|--------|---------|---------|
| **GitHub** | OAuth 2.0 | Nebula secure vault | Auto-refresh |
| **Google Sheets** | OAuth 2.0 | Nebula secure vault | Auto-refresh |
| **CoinGecko** | None (public) | N/A | N/A |
| **DEXScreener** | None (public) | N/A | N/A |
| **Polymarket** | None (public) | N/A | N/A |
| **Ostium** | API Key | Nebula secure vault | Manual rotation |

---

### Security Best Practices

1. **Credentials**
   - Never hardcode API keys in scripts
   - Use Nebula's secure credential storage
   - Rotate keys quarterly (or per provider policy)

2. **API Scopes**
   - Request minimum necessary permissions
   - Google Sheets: Only `spreadsheets` scope (not full Drive)
   - GitHub: Only repo access (not org admin)

3. **Data Privacy**
   - No PII in scripts or logs
   - Trading positions are non-sensitive (personal risk)
   - Google Sheets shared read-only publicly (if desired)

4. **Error Handling**
   - Never expose API keys in error messages
   - Sanitize logs before sharing
   - Use generic error messages in public docs

5. **Rate Limiting**
   - Implement exponential backoff on retries
   - Cache frequently accessed data
   - Respect provider rate limits

---

## Deployment Architecture

### Nebula Environment

**Execution Layer:**
- E2B sandboxes for Python script execution
- Isolated environments per script run
- Pre-installed packages: pandas, numpy, requests, etc.

**Storage Layer:**
- Persistent: Scripts, triggers, memories
- Session: Temporary files in tmp/
- External: Google Sheets, GitHub

**Networking:**
- Outbound HTTPS to all APIs
- Rate limiting handled by Nebula platform
- No inbound connections (security)

---

### GitHub Pages Hosting

**Repository:** dutchiono/trading-battleplans  
**Branch:** main  
**Build:** Jekyll (automatic)  
**CDN:** GitHub's global CDN  
**Custom Domain:** (optional) trading.yourdomain.com

**Deployment Flow:**
```
Local/Nebula
    ↓ (git push)
GitHub Repository (main branch)
    ↓ (auto-trigger)
Jekyll Build
    ↓
GitHub Pages CDN
    ↓
https://dutchiono.github.io/trading-battleplans/
```

**Build Time:** 2-5 minutes  
**Cache TTL:** 10 minutes  
**Uptime:** 99.9% SLA (GitHub)

---

### Google Sheets Dashboard

**Access Control:**
- Owner: dutchiono@gmail.com
- Editors: Nebula service account
- Viewers: (optional) Share link for read-only

**Backup Strategy:**
- Google Sheets version history (30 days)
- Monthly export to CSV (manual)
- Critical: Script-based backup to GitHub (planned)

**Performance:**
- <1000 rows per tab (optimal)
- Minimal heavy formulas (pre-calculate in script)
- Conditional formatting cached

---

## System Maintenance

### Daily
- [ ] Review BMNR position tracker output
- [ ] Check core portfolio performance
- [ ] Scan watchlist for alerts
- [ ] Read daily battleplan
- [ ] Update Google Sheets with trade outcomes

### Weekly
- [ ] Audit script success rates (manage_scripts)
- [ ] Review trigger execution history
- [ ] Analyze closed Polymarket positions
- [ ] Refine edge detection algorithms
- [ ] Update documentation if workflows change

### Monthly
- [ ] Archive old Google Sheets data
- [ ] Rotate API keys (if applicable)
- [ ] Review and optimize slow scripts
- [ ] Backtest volatility scanner (when implemented)
- [ ] Update README with new features

### Quarterly
- [ ] Full system audit
- [ ] Disaster recovery test (restore from SETUP.md)
- [ ] Evaluate new data sources
- [ ] Refactor technical debt
- [ ] Security review

---

## Monitoring & Observability

### Key Metrics

**Script Health:**
- Success rate (target: >95%)
- Average execution time
- Error frequency by type
- API response times

**Trigger Reliability:**
- On-time execution rate
- Missed executions (skipped crons)
- Failure-to-retry rate

**Data Quality:**
- Google Sheets update success rate
- Duplicate entry frequency
- Stale data detection (age >24h)

**Trading Performance:**
- Win rate (Polymarket + stocks/crypto)
- Average edge size vs. realized outcome
- P&L tracking accuracy

### Alerting Strategy

**Critical Alerts (Immediate Action):**
- BMNR position near liquidation
- Script failing for >3 consecutive runs
- Google Sheets authentication expired

**Warning Alerts (Review Soon):**
- API rate limit approaching
- Script success rate <90%
- Data staleness detected

**Info Notifications (FYI):**
- Daily battleplan published
- New high-conviction edge detected
- Weekly performance summary

---

## Disaster Recovery

### Backup Strategy

**Configuration Backup:**
- All documentation in GitHub (SCRIPTS.md, SHEETS.md, SETUP.md)
- Script code in Nebula scripts/
- Trigger definitions in Nebula triggers/

**Data Backup:**
- Google Sheets: Version history + manual CSV exports
- GitHub: Git version control
- Nebula: Platform-managed backups

### Recovery Procedures

**Scenario 1: Nebula Instance Reset**
See [SETUP.md](SETUP.md) for step-by-step restoration.

**Scenario 2: Google Sheets Corruption**
1. Access version history
2. Restore to last known good state
3. Re-run scripts to populate missing data

**Scenario 3: GitHub Repository Deleted**
1. Recreate repository
2. Push from local backup (if available)
3. Regenerate documentation via Nebula
4. Re-enable GitHub Pages

**Scenario 4: API Access Revoked**
1. Re-authenticate via Nebula
2. Update API keys in secure vault
3. Test script execution
4. Resume normal operations

---

## Future Enhancements

### Short Term (1-3 Months)
1. Fix Polymarket edge updater script (0% → 100% success)
2. Implement bi-directional volatility scanner
3. Add Telegram notifications for critical alerts
4. Automate daily battleplan generation (6 AM trigger)
5. Create performance dashboard in Google Sheets

### Medium Term (3-6 Months)
1. Implement machine learning for edge probability estimates
2. Add live Polymarket position tracking via API
3. Create mobile-friendly battleplan format
4. Build historical backtest engine for signals
5. Integrate TradingView for technical analysis overlays

### Long Term (6-12 Months)
1. Automated trade execution (Polymarket, DEXs)
2. Portfolio optimization engine
3. Multi-user support (share battleplans with team)
4. Voice alerts via phone call integration
5. Custom AI agent for strategy research

---

## Related Documentation

- [README.md](README.md) - Quick start and overview
- [SCRIPTS.md](SCRIPTS.md) - Detailed script documentation
- [SHEETS.md](SHEETS.md) - Google Sheets integration
- [SETUP.md](SETUP.md) - Rapid scaffolding guide
- [Index (Battleplan)](index.md) - Daily trading plan

---

## Glossary

**Agent:** Specialized AI assistant within Nebula (e.g., GitHub Agent, Polymarket Agent)  
**Edge:** Difference between market-implied probability and research-based probability  
**P&L:** Profit & Loss  
**Priced In:** Market has already accounted for known information  
**Recipe:** Task template with defined steps for triggers  
**Script:** Python automation code stored in Nebula  
**Trigger:** Scheduled job (cron or event-driven) that executes recipes  

---

**System Version:** 1.0  
**Last Updated:** 2026-02-11  
**Architecture Maintained By:** Nebula AI + dutchiono