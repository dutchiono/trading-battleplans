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

See [SCRIPTS.md](SCRIPTS.md) for complete documentation.

**Active Scripts:**
1. BMNR Short Tracker (100% success)
2. Core Portfolio Tracker (100% success)
3. Watchlist Token Tracker (100% success)
4. Polymarket Edge Updater (0% - needs fixing)
5. Volatility Scanner (0% - not implemented)

---

### 3. Triggers (Scheduling Layer)

**Active:**
- `@trigger:bmnr-short-position-hourly-tracker` - Hourly BMNR monitoring

**Paused:**
- `@trigger:ken-paxton-no-position-hourly-tracker` - Polymarket position

---

### 4. Google Sheets Dashboard

See [SHEETS.md](SHEETS.md) for complete documentation.

**Tabs:**
1. Edge Candidates - Polymarket opportunities
2. Trade Signal Log - Historical signals
3. Performance Dashboard - KPIs and charts

---

### 5. GitHub Pages Documentation

**Repository:** https://github.com/dutchiono/trading-battleplans  
**Live Site:** https://dutchiono.github.io/trading-battleplans/

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
```

For detailed workflows, see full ARCHITECTURE.md in the repository.

---

## Integration Map

| Service | Purpose | Auth | Rate Limits |
|---------|---------|------|-------------|
| Ostium | Perp positions | API Key | TBD |
| CoinGecko | Crypto prices | Public | 10-50/min |
| DEXScreener | DEX data | Public | Generous |
| Polymarket | Prediction markets | Public | 100/min |
| Google Sheets | Dashboard | OAuth 2.0 | 100/100s |
| GitHub | Documentation | OAuth | 5000/hr |

---

## Automation Workflows

### Workflow 1: Hourly Position Check
- Trigger fires every hour
- BMNR script executes
- Position P&L calculated
- Alert if near critical levels

### Workflow 2: Daily Battleplan
- Research earnings, catalysts, crypto momentum
- Rank opportunities by risk/reward
- Generate Markdown battleplan
- Push to GitHub Pages
- Live at public URL

### Workflow 3: Edge Detection
- Scan Polymarket for active markets
- Research external probability sources
- Calculate edge (research vs market odds)
- Update Google Sheets
- Alert on high-conviction opportunities

---

## Security & Authentication

**Methods:**
- GitHub: OAuth 2.0 (auto-refresh)
- Google Sheets: OAuth 2.0 (auto-refresh)
- Ostium: API Key (secure vault)
- Public APIs: No authentication required

**Best Practices:**
- Never hardcode credentials
- Use Nebula secure storage
- Rotate keys quarterly
- Minimum necessary scopes
- Sanitize logs

---

## Deployment Architecture

**Nebula:**
- E2B sandboxes for script execution
- Persistent storage for scripts/triggers
- Session storage for temp files

**GitHub Pages:**
- Jekyll auto-build
- Global CDN
- 2-5 min deploy time
- 99.9% uptime SLA

**Google Sheets:**
- Cloud-hosted dashboard
- Real-time collaboration
- Version history (30 days)
- OAuth service account access

---

## Disaster Recovery

See [SETUP.md](SETUP.md) for complete restoration procedures.

**Backups:**
- Documentation: GitHub repository
- Scripts: Nebula platform
- Data: Google Sheets version history
- Triggers: Nebula configuration

**Recovery Time:**
- Scripts: 15-30 minutes
- Triggers: 10 minutes
- Documentation: 5 minutes (git clone)
- Google Sheets: Instant (version restore)

---

## Future Enhancements

**Short Term:**
- Fix Polymarket edge updater
- Implement volatility scanner
- Add Telegram notifications
- Automate daily battleplan (6 AM)

**Medium Term:**
- ML-based edge estimates
- Live position tracking
- Mobile-friendly format
- Historical backtesting

**Long Term:**
- Automated trade execution
- Portfolio optimization
- Multi-user support
- Voice alerts

---

## Related Documentation

- [README.md](README.md) - Quick start
- [SCRIPTS.md](SCRIPTS.md) - Script details
- [SHEETS.md](SHEETS.md) - Dashboard guide
- [SETUP.md](SETUP.md) - Rapid restoration
- [index.md](index.md) - Daily battleplan

---

**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI + dutchiono