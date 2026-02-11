# Daily Trading Battleplans

Automated market analysis and trade planning system powered by Nebula AI.

## Quick Links

- **[Today's Battleplan](https://dutchiono.github.io/trading-battleplans/)** - Live GitHub Pages site
- **[System Architecture](ARCHITECTURE.md)** - How everything works
- **[Scripts Documentation](SCRIPTS.md)** - All automation scripts
- **[Setup Guide](SETUP.md)** - Rapid scaffolding instructions

## What This Is

Daily trading battleplans combining:
- Pre-market catalyst scanning
- Technical analysis and momentum detection  
- Position tracking (BMNR shorts, core portfolio, watchlists)
- Risk-ranked opportunities with entry/exit levels
- Honest takes on what to trade vs. what to skip

## System Components

### Active Scripts
1. **BMNR Short Tracker** - Monitors Ostium short positions with P&L
2. **Core Portfolio Tracker** - BTC, ETH, SOL, XMR, MONAD prices
3. **Watchlist Token Tracker** - Custom tokens (Solana + Base)
4. **Polymarket Edge Detector** - Mispriced prediction markets
5. **Bi-Directional Volatility Scanner** - Pump/dump detection

### Active Triggers
- Hourly BMNR position checks
- (More triggers documented in ARCHITECTURE.md)

### Integrations
- Google Sheets: Edge Candidates tracking
- Polymarket Gamma API
- CoinGecko API
- Multiple crypto data sources

## Daily Workflow

1. Pre-market: Nebula generates battleplan
2. Review: Check opportunities, rankings, risk levels
3. Trade: Execute highest-conviction setups
4. Monitor: Hourly position trackers run automatically
5. Update: Daily battleplan pushed to GitHub Pages

## If Nebula Resets

See [SETUP.md](SETUP.md) for complete restoration instructions.

---

**Live Site:** https://dutchiono.github.io/trading-battleplans/

Updated: 2026-02-11