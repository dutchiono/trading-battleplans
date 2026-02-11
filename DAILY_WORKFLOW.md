# Daily Battleplan Update Workflow

Complete guide for generating and publishing daily trading battleplans to GitHub Pages.

## Overview

**Purpose:** Generate fresh daily battleplans with market analysis, catalyst tracking, and risk-ranked opportunities.

**Output:** Live GitHub Pages site at https://dutchiono.github.io/trading-battleplans/

**Frequency:** Daily (recommended 6:00 AM EST, before market open)

**Time Required:** 5-10 minutes manual, <2 minutes automated

---

## Manual Workflow (Current)

### Step 1: Morning Request (5-7 AM EST)

**Command to Nebula:**
```
"Generate today's trading battleplan for [DAY], [DATE]"

Example:
"Generate today's trading battleplan for Tuesday, February 11, 2026"
```

### Step 2: Review & Edit (Optional)

Review the battleplan for accuracy and personal preferences.

### Step 3: Push to GitHub Pages

**Option A: Nebula Handles It (Recommended)**
```
"Push this battleplan to GitHub Pages"
```

**Option B: Manual Push**
```bash
git clone https://github.com/dutchiono/trading-battleplans.git
cd trading-battleplans
# Edit index.md
git add index.md
git commit -m "feat: update battleplan for [DATE]"
git push origin main
```

### Step 4: Verify Live Site

https://dutchiono.github.io/trading-battleplans/

### Step 5: Share & Use

Access throughout the day for entry/exit levels and risk management.

---

## Automated Workflow (Planned)

### Create Daily Trigger

```python
write_task(
    title='Daily Trading Battleplan Generation',
    description='Generate and publish daily battleplan at 6:00 AM EST',
    steps=[...]
)

manage_triggers(
    action='create',
    name='Daily Battleplan Generator',
    cron_expression='0 11 * * *',  # 6 AM EST = 11 AM UTC
    recipe='<path_to_task_recipe>'
)
```

**Automatic Process:**
- 6:00 AM EST: Trigger fires
- 6:01-6:05 AM: Research and generate
- 6:05-6:07 AM: Push to GitHub
- 6:07 AM: Send confirmation
- You wake up: Battleplan ready

---

## Battleplan Template Structure

### Required Sections

1. **Title & Date**
2. **Current Positions** (BMNR, Polymarket, etc.)
3. **New Opportunities** (3-5 ranked)
4. **Master Gameplan** (pre-market, open, mid-day, close)
5. **Risk Management Rules**
6. **Best Play Ranking**
7. **Morning Checklist**

---

## Customization Options

### Adjust Research Sources
```
"Include options flow data in today's battleplan"
"Focus on crypto-only opportunities"
"Add Polymarket prediction markets"
```

### Adjust Risk Profile
```
"Make today's battleplan more conservative"
"Include high-risk/high-reward plays"
"Skip all after-hours plays"
```

### Change Format
```
"Generate a condensed battleplan - top 3 only"
"Add technical chart analysis"
```

---

## Integration with Other Tools

### Google Sheets Sync
1. Review new opportunities
2. Update Status (WATCHING/ENTERED/SKIPPED)
3. Log trades in Trade Signal Log

### Position Monitoring
- BMNR tracker runs hourly
- Check status: "What's my BMNR P&L?"

### Telegram Notifications (Planned)
- Daily battleplan published
- High-conviction opportunities
- Position alerts

---

## Version Control & History

### Archiving Old Battleplans

**Git History (Automatic):**
https://github.com/dutchiono/trading-battleplans/commits/main/index.md

**Manual Archive:**
```bash
mkdir battleplans_archive
cp index.md battleplans_archive/2026-02-11_battleplan.md
git add battleplans_archive/
git commit -m "archive: battleplan for [DATE]"
```

### Performance Tracking
- Weekly review of past battleplans
- Monthly win rate analysis
- Strategy refinement

---

## Troubleshooting

### Battleplan Generation Fails
- Check API connectivity
- Retry with simpler request
- Use cached data

### GitHub Push Fails
- Re-authenticate GitHub
- Verify repository access
- Manual push as fallback

### GitHub Pages Not Updating
- Wait 5 minutes (build time)
- Check GitHub Actions
- Clear browser cache
- Verify Jekyll front matter

### Missing or Stale Data
- Verify API access
- Double-check dates manually
- Add data verification disclaimer

---

## Best Practices

### Timing
- 6:00 AM: Auto-generate
- 7:00 AM: Review
- 8:30 AM: Set alerts
- 9:30 AM: Execute

### Content Quality
**Always include:**
- Specific entry/exit/stop levels
- Position sizing
- Risk assessment
- Morning checklist

### Consistency
1. Generate (6 AM)
2. Review (7 AM)
3. Set alerts (8:30 AM)
4. Execute (9:30 AM+)
5. Log outcomes (EOD)
6. Weekly review (Sunday)

---

## Future Enhancements

### Short Term
- [ ] Automated 6 AM trigger
- [ ] Telegram notifications
- [ ] Auto-archive to folder
- [ ] Link to Google Sheets

### Medium Term
- [ ] Live price widgets
- [ ] TradingView embeds
- [ ] Mobile-optimized layout
- [ ] Voice summary

### Long Term
- [ ] Multi-day outlook
- [ ] Performance analytics
- [ ] AI strategy refinement
- [ ] Community sharing

---

## Example: Full Daily Workflow

**6:00 AM** - Generate
**6:05 AM** - Push to GitHub
**6:10 AM** - Review live site
**7:00 AM** - Pre-market actions
**8:30 AM** - Set alerts
**9:30 AM** - Execute trades
**4:00 PM** - Log outcomes
**Evening** - Archive & review

---

## Quick Reference Commands

```
"Generate today's trading battleplan"
"Push this battleplan to GitHub Pages"
"Is today's battleplan live?"
"Update the BMNR section with current price"
"Archive today's battleplan"
```

---

## Related Documentation

- [README.md](README.md) - System overview
- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [SCRIPTS.md](SCRIPTS.md) - Automation scripts
- [SETUP.md](SETUP.md) - Setup guide
- [SHEETS.md](SHEETS.md) - Dashboard integration

---

**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI + dutchiono