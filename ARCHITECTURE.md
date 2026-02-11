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
8. [Privacy & Data Separation](#privacy--data-separation)

---

## System Overview

### Mission
Automate trading intelligence, opportunity detection, and market analysis across crypto and prediction markets while maintaining a public-facing daily battleplan for strategic insights and a private position tracker for personal trade management.

### Design Philosophy
- **Modular**: Each script is independent and reusable
- **Event-Driven**: Triggers orchestrate scheduled execution
- **Observable**: All data flows through central dashboard (Google Sheets)
- **Resilient**: Graceful failure handling and retry logic
- **Documented**: Every component has clear purpose and usage instructions
- **Privacy-First**: Clear separation between public analysis and private positions

---

## Privacy & Data Separation

### Public vs Private Data

#### PUBLIC (GitHub Pages - trading-battleplans repository)
**What's Published:**
- Market opportunity analysis
- Catalyst calendar (earnings, FDA, economic data)
- Technical setup ideas with entry/exit levels
- Post-spike short candidates from scanner
- Risk management guidelines
- Educational trading patterns
- Forward calendar of upcoming events

**Format:** "Here are today's setups" (advice/education, not personal trades)

**Audience:** Anyone following the GitHub Pages site

#### PRIVATE (Nebula workspace only - NEVER committed to GitHub)
**What Stays Private:**
- Your actual entry prices
- Your position sizes
- Your realized/unrealized P&L
- Your stop loss levels (actual positions)
- Your personal trade timing and execution

**Storage:** `data/private_positions.json` (gitignored)

**Format:** Personal trade journal with actual performance tracking

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
Output: PRIVATE data file only (data/private_positions.json)
```

**Data Flow:**
```
Ostium API → Script → Position Analysis → PRIVATE STORAGE ONLY
```

**Key Metrics:**
- Entry price
- Current P&L percentage
- Liquidation distance
- Exit recommendations

**IMPORTANT:** This data is NOT published to GitHub. Used only for private position tracking.

---

#### 2.2 Core Portfolio Tracker
```
Location: scripts/coingecko/core_portfolio_tracker.py
Purpose: Track BTC, ETH, SOL, XMR, MONAD prices
Frequency: On-demand / Hourly
Success Rate: 100%
Output: PRIVATE data file only
```

**Data Flow:**
```
CoinGecko API → Script → Price Aggregation → PRIVATE STORAGE ONLY
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
Output: PRIVATE data file only
```

**Data Flow:**
```
DEXScreener API → Script → Price Analysis → PRIVATE STORAGE ONLY
```

**Alert Conditions:**
- +20% in 1h (pump alert)
- -15% in 1h (dump alert)
- 3x volume spike
- Liquidity <$100K (rug risk)

---

#### 2.4 Biotech Spike Scanner
```
Location: scripts/python/biotech_spike_scanner.py
Purpose: Identify +40% biotech moves for post-spike short analysis
Frequency: Daily (automated) / On-demand
Success Rate: 100%
Output: PUBLIC opportunities (anonymized, no position data)
```

**Data Flow:**
```
Finviz/Web Search → Scanner → Opportunity Analysis → PUBLIC BATTLEPLAN
```

**What Gets Published:**
- Ticker and catalyst description
- Technical levels and crowd sentiment
- Risk assessment and setup ideas
- "Priced in" analysis

**What NEVER Gets Published:**
- Whether you actually entered the trade
- Your entry price or position size
- Your P&L on the trade

---

### 3. Data Storage

#### 3.1 Public Data (GitHub Repository)
**Location:** `dutchiono/trading-battleplans`
```
├── index.md              # Current day's public battleplan
├── archive/              # Historical public battleplans
│   └── YYYY-MM-DD.md
├── ARCHITECTURE.md       # This file
├── DAILY_WORKFLOW.md     # Workflow documentation
└── data/
    └── forward_calendar.json  # Upcoming catalysts
```

#### 3.2 Private Data (Nebula Workspace - Gitignored)
**Location:** Nebula file storage (never pushed to GitHub)
```
data/
├── private_positions.json     # Your actual trades
├── position_history.json      # Historical P&L
└── trade_journal.json         # Trade notes and learnings
```

**`.gitignore` Contents:**
```
data/private_*.json
data/position_*.json
data/trade_*.json
```

---

### 4. GitHub Integration

#### 4.1 Repository Structure
```
dutchiono/trading-battleplans/
├── index.md                    # PUBLIC: Current battleplan
├── archive/                    # PUBLIC: Historical battleplans
├── ARCHITECTURE.md             # PUBLIC: System docs
├── DAILY_WORKFLOW.md           # PUBLIC: Process docs
├── SETUP.md                    # PUBLIC: Setup guide
├── SHEETS.md                   # PUBLIC: Sheets integration
└── data/
    └── forward_calendar.json   # PUBLIC: Upcoming events
```

#### 4.2 GitHub Pages Deployment
- **URL:** `https://dutchiono.github.io/trading-battleplans/`
- **Trigger:** Push to `main` branch
- **Build Time:** 2-5 minutes
- **Theme:** Minimal (markdown rendering)

---

## Data Flow

### Daily Battleplan Generation (8:00 AM ET)

```
┌─────────────────────────────────────────────────────────────────┐
│                   TRIGGER: Daily 8 AM ET                        │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 1: Gather Market Intelligence (PUBLIC DATA ONLY)          │
│  ─────────────────────────────────────────────────────────      │
│  • Search today's catalysts (earnings, FDA, economic data)      │
│  • Run Biotech Spike Scanner for post-spike opportunities       │
│  • Load forward calendar for upcoming events                    │
│  • Web search for market sentiment and news                     │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 2: Generate PUBLIC Battleplan (index.md)                  │
│  ─────────────────────────────────────────────────────────      │
│  • Format as educational/advisory content                       │
│  • Include: catalysts, setups, technical levels                 │
│  • Add: risk warnings, "priced in" analysis                     │
│  • Provide: entry/exit ideas (generic, not YOUR trades)         │
│  • NO personal position data, NO P&L, NO actual entries         │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 3: Update PRIVATE Position Tracker (Nebula only)          │
│  ─────────────────────────────────────────────────────────      │
│  • Run BMNR Short Tracker (Ostium API)                          │
│  • Run Core Portfolio Tracker (CoinGecko)                       │
│  • Calculate current P&L on all positions                       │
│  • Update data/private_positions.json                           │
│  • NEVER commit this file to GitHub                             │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 4: Publish PUBLIC Battleplan to GitHub                    │
│  ─────────────────────────────────────────────────────────      │
│  • Archive yesterday's index.md to archive/YYYY-MM-DD.md        │
│  • Update index.md with today's PUBLIC battleplan               │
│  • Commit and push to main branch                               │
│  • GitHub Pages auto-deploys (2-5 min)                          │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  Step 5: Notify User in Nebula                                  │
│  ─────────────────────────────────────────────────────────      │
│  • Summary of public battleplan published                       │
│  • Private position update (in Nebula chat only)                │
│  • Link to live GitHub Pages                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Integration Map

### External APIs & Services

#### Trading Data
- **Ostium API**: Perpetual positions (PRIVATE tracking only)
- **CoinGecko API**: Crypto prices (PRIVATE portfolio only)
- **DEXScreener API**: Token data (PRIVATE watchlist only)
- **Polymarket API**: Prediction markets (PRIVATE edge detection only)

#### Market Intelligence (PUBLIC)
- **Finviz**: Biotech screener for spike detection
- **Web Search**: News, catalysts, FDA calendar
- **Benzinga Calendar**: Earnings and events
- **TradingView**: Chart links (PUBLIC battleplan)

#### Infrastructure
- **GitHub API**: Repository management and deployment
- **GitHub Pages**: Public battleplan hosting
- **Nebula**: Private data storage and automation

---

## Automation Workflows

### 1. Daily Battleplan Trigger
```yaml
Name: daily-trading-battleplan-generator
Schedule: 0 8 * * * (8:00 AM ET daily)
Task Recipe: recipes/file_0698be257b2d7931800072758e2434b7/TASK.md
Output: 
  - PUBLIC: dutchiono/trading-battleplans/index.md
  - PRIVATE: data/private_positions.json (Nebula only)
```

### 2. Hourly Position Trackers (PRIVATE)
```yaml
Name: bmnr-short-position-hourly-tracker
Schedule: 0 * * * * (Every hour)
Output: data/private_positions.json (Nebula only, NEVER GitHub)
```

### 3. On-Demand Scripts
- Biotech Spike Scanner (manual execution for real-time opportunities)
- Core Portfolio Tracker (check crypto holdings)
- Watchlist Token Tracker (monitor specific tokens)

---

## Security & Authentication

### API Keys & Credentials
- **Storage:** Nebula secure credential vault
- **Access:** Automatic injection via run_action()
- **Rotation:** Manual (as needed per API provider)

### GitHub Authentication
- **Method:** Pipedream OAuth (apn_XehbgoX)
- **Scope:** repo, workflow, read:org
- **Auto-refresh:** Handled by Nebula

### Data Privacy
- **Public Data:** Carefully curated, educational content only
- **Private Data:** Never leaves Nebula workspace
- **Git Configuration:** `.gitignore` prevents accidental commits

---

## Deployment Architecture

### Production Stack
```
┌─────────────────────────────────────────────────────────┐
│                    Nebula AI Platform                   │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Scripts (Python)                                 │  │
│  │  • Position Trackers (PRIVATE)                    │  │
│  │  • Biotech Scanner (PUBLIC)                       │  │
│  │  • Portfolio Monitors (PRIVATE)                   │  │
│  └────────────────┬─────────────────────────────────┘  │
│                   │                                     │
│  ┌────────────────▼─────────────────────────────────┐  │
│  │  Triggers (Cron)                                  │  │
│  │  • Daily Battleplan (8 AM ET)                     │  │
│  │  • Hourly Position Updates (PRIVATE)              │  │
│  └────────────────┬─────────────────────────────────┘  │
│                   │                                     │
│  ┌────────────────▼─────────────────────────────────┐  │
│  │  Data Storage                                     │  │
│  │  • Private: data/private_*.json (gitignored)      │  │
│  │  • Public: Prepared for GitHub push               │  │
│  └────────────────┬─────────────────────────────────┘  │
└────────────────────┼─────────────────────────────────────┘
                     │
                     │ GitHub API Push (PUBLIC only)
                     ▼
┌─────────────────────────────────────────────────────────┐
│           GitHub Repository (PUBLIC)                    │
│  dutchiono/trading-battleplans                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  • index.md (today's PUBLIC battleplan)           │  │
│  │  • archive/ (historical PUBLIC battleplans)       │  │
│  │  • Documentation (ARCHITECTURE, WORKFLOW, etc)    │  │
│  └────────────────┬─────────────────────────────────┘  │
└────────────────────┼─────────────────────────────────────┘
                     │
                     │ GitHub Pages Auto-Deploy
                     ▼
┌─────────────────────────────────────────────────────────┐
│         GitHub Pages (Public Website)                   │
│  https://dutchiono.github.io/trading-battleplans/       │
│  • Educational trading setups                           │
│  • Market analysis and opportunities                    │
│  • NO personal position data                            │
└─────────────────────────────────────────────────────────┘
```

---

## Best Practices

### For Public Battleplan Content
✅ **DO:**
- Provide educational setup ideas
- Include technical levels and risk analysis
- Share market research and catalyst information
- Offer risk management guidelines
- Use language like "Consider...", "Watch for...", "Setup available at..."

❌ **DON'T:**
- Share your actual entry prices
- Publish your position sizes or P&L
- Say "I entered at..." or "My position..."
- Include stop loss levels of actual trades
- Reveal your personal trade timing

### For Private Position Tracking
✅ **DO:**
- Track all metrics in private data files
- Use scripts to auto-update P&L
- Store trade journal entries locally
- Review performance in Nebula chat
- Keep detailed notes for strategy improvement

❌ **DON'T:**
- Commit private data files to GitHub
- Share position details in public documentation
- Mix private and public data in same file
- Accidentally archive private data

---

## Monitoring & Observability

### Success Metrics
- **Script Success Rate**: Track via manage_scripts action
- **Trigger Execution**: Monitor via manage_triggers action
- **GitHub Pages Uptime**: Visual check of deployed site
- **API Rate Limits**: Monitor usage to avoid throttling

### Error Handling
- Graceful degradation (skip failed components, continue workflow)
- Retry logic for transient API failures
- Detailed error messages in Nebula chat
- Fallback to manual execution if automation fails

---

**Last Updated:** February 11, 2026
**Next Review:** When adding new integrations or workflows
