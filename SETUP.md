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

See [SCRIPTS.md](SCRIPTS.md) for complete script documentation.

**Quick Script Recreation:**

1. **BMNR Short Tracker** - Ostium position monitoring
2. **Core Portfolio Tracker** - BTC, ETH, SOL, XMR, MONAD
3. **Watchlist Token Tracker** - Custom tokens (Solana, Base)
4. **Polymarket Edge Updater** - Needs debugging
5. **Volatility Scanner** - Not yet implemented

Use `manage_scripts(action='create', ...)` for each script.

---

## Step 3: Restore Triggers

### BMNR Hourly Tracker

```python
manage_triggers(
    action='create',
    name='BMNR Short Position Hourly Tracker',
    description='Hourly monitoring of BMNR short positions',
    trigger_type='cron',
    cron_expression='0 * * * *',
    recipe='<path_to_task_recipe>'
)
```

---

## Step 4: Reconnect APIs

### OAuth Services
- **GitHub:** `account_id: apn_XehbgoX`
- **Google Sheets:** Re-authenticate if needed

### Public APIs (No Auth)
- CoinGecko
- DEXScreener
- Polymarket Gamma

### Private APIs
- Ostium: Store API key in Nebula vault

---

## Step 5: Verify Google Sheets

1. Locate your Google Sheets ID
2. Confirm tabs: Edge Candidates, Trade Signal Log, Performance Dashboard
3. Test write access
4. Verify formulas intact

See [SHEETS.md](SHEETS.md) for detailed structure.

---

## Step 6: Test Execution

```python
# Test individual scripts
manage_scripts(action='run', script_path='scripts/ostium/bmnr_short_tracker.py')
manage_scripts(action='run', script_path='scripts/coingecko/core_portfolio_tracker.py')

# Test triggers
manage_triggers(action='get', trigger_slug='bmnr-short-position-hourly-tracker')

# Generate test battleplan
# Ask Nebula: "Generate tomorrow's trading battleplan"
```

---

## Step 7: Enable GitHub Pages

1. Go to: https://github.com/dutchiono/trading-battleplans/settings/pages
2. Source: Deploy from branch
3. Branch: main
4. Folder: / (root)
5. Save

**Verify:** https://dutchiono.github.io/trading-battleplans/

---

## Step 8: Validate Workflows

### Position Monitoring
- [ ] BMNR tracker runs hourly
- [ ] P&L calculated correctly
- [ ] Recommendations generated

### Portfolio Tracking
- [ ] Core portfolio fetches prices
- [ ] All 5 assets tracked
- [ ] Changes displayed

### Battleplan Generation
- [ ] Research multiple sources
- [ ] Opportunities ranked
- [ ] Pushed to GitHub Pages
- [ ] Live URL accessible

### Google Sheets
- [ ] Can read data
- [ ] Can write data
- [ ] Formulas preserved

---

## Common Issues & Solutions

### Script Execution Fails
**Solution:** Verify script exists with `manage_scripts(action='list')`

### Trigger Not Firing
**Solution:** Check `is_active: true` and verify cron expression

### GitHub Pages 404
**Solution:** Ensure index.md exists, wait 5 minutes for build

### Google Sheets Auth Error
**Solution:** Re-authenticate via Nebula OAuth flow

### API Rate Limiting
**Solution:** Add exponential backoff, reduce frequency

---

## Recovery Time Objectives

| Component | Target Recovery Time |
|-----------|---------------------|
| Scripts | 30 minutes |
| Triggers | 15 minutes |
| API Connections | 10 minutes |
| GitHub Pages | 5 minutes |
| Google Sheets | Instant |
| **Total** | **60 minutes** |

---

## Post-Setup Verification

```python
# 1. List all scripts (expect 5)
manage_scripts(action='list')

# 2. List all triggers (expect 1-2)
manage_triggers(action='list')

# 3. Test BMNR tracker
manage_scripts(action='run', script_path='scripts/ostium/bmnr_short_tracker.py')

# 4. Test core portfolio
manage_scripts(action='run', script_path='scripts/coingecko/core_portfolio_tracker.py')

# 5. Generate test battleplan
# Ask Nebula: "Generate a test battleplan"

# 6. Verify GitHub Pages
# Visit: https://dutchiono.github.io/trading-battleplans/
```

---

## Next Steps

1. Customize token addresses in Watchlist Tracker
2. Fix Polymarket Edge Updater (0% success)
3. Implement Volatility Scanner
4. Add Telegram notifications
5. Create automated 6 AM battleplan trigger

---

## Related Documentation

- [README.md](README.md) - Quick start
- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [SCRIPTS.md](SCRIPTS.md) - Script details
- [SHEETS.md](SHEETS.md) - Dashboard guide

---

**Last Updated:** 2026-02-11  
**Author:** Nebula AI + dutchiono