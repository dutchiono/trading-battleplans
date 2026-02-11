# Privacy & Data Separation

## Overview
This repository contains **PUBLIC trading analysis and setup ideas** only. Personal position data, entry prices, and P&L tracking are stored privately in Nebula and **NEVER** committed to this repository.

---

## What's Public (In This Repository)

✅ **Market Analysis**
- Daily catalyst calendar (earnings, FDA, economic data)
- Post-spike short opportunities with risk analysis
- Technical setup ideas with generic entry/exit levels
- Forward calendar of upcoming events
- Educational content and trading patterns

✅ **Format**
- Advisory language: "Consider...", "Watch for...", "Setup available at..."
- Generic technical levels for educational purposes
- Risk assessments and "priced in" analysis
- Links to charts, news, and research sources

✅ **Audience**
- Anyone following the GitHub Pages site
- Traders looking for setup ideas and market intelligence
- Educational content for learning trading strategies

---

## What's Private (NOT In This Repository)

❌ **Personal Trading Data**
- Your actual entry prices
- Your position sizes (number of shares/contracts)
- Your realized and unrealized P&L
- Your stop loss levels on active positions
- Your trade timing and execution details
- Your personal trade journal and notes

❌ **Storage Location**
- Stored in Nebula workspace only
- File path: `data/private_positions.json` (gitignored)
- Never committed to GitHub
- Never visible on GitHub Pages

---

## Technical Implementation

### .gitignore Configuration
The following patterns prevent private data from being committed:

```
# Private position data - NEVER commit these files
data/private_*.json
data/position_*.json
data/trade_*.json
```

### Automated Workflow Separation

The daily battleplan automation generates **two separate outputs**:

#### 1. PUBLIC Battleplan (index.md)
- Generated from market research and opportunity scanning
- Contains setup ideas and technical analysis
- Published to GitHub and deployed to GitHub Pages
- Updated daily at 8:00 AM ET

#### 2. PRIVATE Position Tracker (Nebula only)
- Generated from position tracking scripts (BMNR Short Tracker, Portfolio Tracker, etc.)
- Contains your actual trades and P&L
- Stored in `data/private_positions.json`
- Updated hourly (for active positions)
- **NEVER committed to GitHub**

---

## Safety Checks

### Pre-Commit Validation
Before publishing to GitHub, the automation checks for accidental private data leaks:

**Red Flags** (will abort publish if detected):
- "Entry: $X.XX" (specific entry prices)
- "Current: $X.XX" (current position prices)
- "P&L" or "profit/loss" mentions with dollar amounts
- "My position" or "I entered at..."
- "Liquidation: $X.XX" (specific liquidation prices)
- "Position size: X shares" (actual position sizes)

### Manual Review Guidelines
If manually updating the battleplan:

❌ **Don't Write:**
- "I entered BMNR short at $20.12"
- "My current P&L is +2.44%"
- "My stop loss is at $21.50"
- "I'm holding 150 shares"

✅ **Do Write:**
- "BMNR offers a short setup around technical resistance"
- "Consider entries on weakness near $20.00"
- "Risk management suggests stops above recent highs"
- "Position sizing: 25% max for high-risk trades"

---

## Data Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│              Nebula Automation (8 AM ET)                │
└─────────────────┬───────────┬───────────────────────────┘
                  │           │
                  │           │
        ┌─────────▼─────┐    └──────────▼────────────┐
        │  PUBLIC Data  │      │   PRIVATE Data      │
        │  Generation   │      │   Tracking          │
        └───────┬───────┘      └──────────┬──────────┘
                │                          │
                │                          │
    ┌───────────▼────────────┐  ┌─────────▼──────────────┐
    │  Market Intelligence   │  │  Position Scripts      │
    │  • Catalysts           │  │  • BMNR Tracker        │
    │  • Spike Scanner       │  │  • Portfolio Tracker   │
    │  • Forward Calendar    │  │  • P&L Calculations    │
    └───────┬────────────────┘  └─────────┬──────────────┘
            │                             │
            │                             │
    ┌───────▼────────────┐      ┌────────▼───────────────┐
    │  index.md          │      │  private_positions.json│
    │  (PUBLIC)          │      │  (PRIVATE, gitignored) │
    └───────┬────────────┘      └────────────────────────┘
            │
            │
    ┌───────▼────────────┐
    │  GitHub Commit     │
    │  & Pages Deploy    │
    └────────────────────┘
```

---

## For Repository Contributors

If you're contributing to this repository or modifying the automation:

### Rules
1. **Never commit files matching:** `data/private_*.json`, `data/position_*.json`, `data/trade_*.json`
2. **Always use advisory language** in battleplan content (not "I" or "my")
3. **Review diffs before committing** to ensure no personal data leaked
4. **Test locally** before pushing changes to automation scripts

### Adding New Features
- New **market analysis** features → Add to public battleplan generation
- New **position tracking** features → Add to private data tracking, ensure gitignored
- Always maintain clear separation between public and private code paths

---

## Questions & Support

### Why This Separation?
- **Privacy:** Your trading performance and positions remain confidential
- **Education:** Public battleplans provide value to others without revealing your edge
- **Security:** Prevents inadvertent disclosure of sensitive trading information

### What If Private Data Was Accidentally Committed?
1. **Immediately contact repository owner**
2. **Force-push with history rewrite** to remove sensitive commit
3. **Review and strengthen safety checks**
4. **Consider rotating any exposed API keys or credentials**

### How to Verify Separation
```bash
# Check that private files are gitignored
git status
# Should NOT show: data/private_*.json, data/position_*.json, data/trade_*.json

# Verify .gitignore is working
touch data/private_test.json
git status
# File should not appear in untracked files
rm data/private_test.json
```

---

## Version History

- **v2.0 (Feb 11, 2026):** Added privacy separation between public battleplan and private position tracking
- **v1.0 (Feb 10, 2026):** Initial automated battleplan system

---

**Last Updated:** February 11, 2026  
**For Questions:** Review ARCHITECTURE.md and DAILY_WORKFLOW.md for technical details
