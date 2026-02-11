# Google Sheets Integration

Documentation for the Google Sheets integration used to track Polymarket edge opportunities and trading signals.

## Overview

The trading system uses Google Sheets as a central dashboard for:
1. **Polymarket Edge Candidates** - Mispriced prediction markets
2. **Trade Signal Logs** - Historical signal tracking
3. **Performance Metrics** - Backtest results and live performance

---

## Sheet Structure

### Main Spreadsheet
**Name:** Trading Dashboard (or similar)  
**URL:** (Add your Google Sheets URL here)

### Tabs/Worksheets

#### 1. Edge Candidates
Primary sheet for Polymarket edge detection opportunities.

**Columns:**

| Column | Header | Data Type | Description | Formula/Source |
|--------|--------|-----------|-------------|----------------|
| A | Date Added | Date | When edge was detected | `=NOW()` (auto) |
| B | Market Title | Text | Full market question | Polymarket API |
| C | Category | Text | Sports, Politics, Crypto, etc. | Polymarket API |
| D | Current Yes Price | Number | Yes share price ($0.00-$1.00) | Polymarket API |
| E | Current No Price | Number | No share price ($0.00-$1.00) | Polymarket API |
| F | Implied Probability | Percentage | Market-implied probability | `=D2` (Yes price) |
| G | Research Probability | Percentage | External research estimate | Web scraping/analysis |
| H | Edge | Percentage | Difference (Research - Market) | `=G2-F2` |
| I | Edge Size | Text | Small/Medium/Large/Huge | `=IF(ABS(H2)<5%,"Small",IF(ABS(H2)<10%,"Medium",IF(ABS(H2)<20%,"Large","Huge")))` |
| J | Liquidity | Currency | Total liquidity available | Polymarket API |
| K | Volume 24h | Currency | 24h trading volume | Polymarket API |
| L | Resolution Date | Date | When market closes | Polymarket API |
| M | Days to Close | Number | Time remaining | `=L2-TODAY()` |
| N | Risk Score | Number | Composite risk (1-10) | Calculated in script |
| O | Recommendation | Text | BUY YES / BUY NO / PASS | Logic in script |
| P | Position Size | Currency | Suggested $ amount | Based on Kelly Criterion |
| Q | Market URL | URL | Link to Polymarket | Polymarket API |
| R | Notes | Text | Manual research notes | User input |
| S | Status | Dropdown | Watching / Entered / Closed | User selection |
| T | Entry Price | Number | Actual entry price | User input |
| U | Exit Price | Number | Actual exit price | User input |
| V | Realized P&L | Currency | Profit/Loss | `=(U2-T2)*shares` |

**Conditional Formatting:**
- Edge >10%: Green background
- Edge <-10%: Red background
- Risk Score >7: Orange text
- Days to Close <7: Yellow background
- Status "Entered": Bold font

---

#### 2. Trade Signal Log
Historical record of all trading signals (stocks, crypto, prediction markets).

**Columns:**

| Column | Header | Data Type | Description |
|--------|--------|-----------|-------------|
| A | Date | Date | Signal generation date |
| B | Asset | Text | Ticker or token |
| C | Asset Type | Dropdown | Stock/Crypto/Polymarket/Options |
| D | Signal Type | Dropdown | LONG/SHORT/BUY YES/BUY NO |
| E | Entry Price | Number | Recommended entry |
| F | Target Price | Number | Take-profit level |
| G | Stop Loss | Number | Risk management |
| H | Risk/Reward | Number | R:R ratio |
| I | Conviction | Dropdown | Low/Medium/High |
| J | Catalyst | Text | Reason for signal |
| K | Entered? | Checkbox | Did you take the trade? |
| L | Actual Entry | Number | Your entry price |
| M | Actual Exit | Number | Your exit price |
| N | Outcome | Dropdown | Win/Loss/Breakeven/Open |
| O | P&L | Currency | Realized profit/loss |
| P | Notes | Text | Post-trade review |

**Key Metrics (Calculated at Top):**
- Total Signals: `=COUNTA(B:B)-1`
- Win Rate: `=COUNTIF(N:N,"Win")/COUNTIF(K:K,TRUE)`
- Average P&L: `=AVERAGE(O:O)`
- Best Trade: `=MAX(O:O)`
- Worst Trade: `=MIN(O:O)`

---

#### 3. Performance Dashboard
Visual KPIs and charts.

**Sections:**

**A. Summary Stats (Top Left)**
- Total Trades
- Win Rate (%)
- Average R:R
- Total P&L
- Sharpe Ratio (if calculated)

**B. Charts**
1. **Equity Curve** (Line Chart)
   - X-axis: Date
   - Y-axis: Cumulative P&L

2. **Win Rate by Asset Type** (Pie Chart)
   - Stocks, Crypto, Polymarket, Options

3. **P&L Distribution** (Histogram)
   - Bins: <-$500, -$500 to $0, $0 to $500, >$500

4. **Monthly Performance** (Bar Chart)
   - X-axis: Month
   - Y-axis: Total P&L

**C. Breakdown Tables**
- By Asset Type
- By Signal Type (LONG vs SHORT)
- By Conviction Level
- By Catalyst Type

---

## Script Integration

### Authentication

**Method:** OAuth 2.0 via Nebula  
**Scopes Required:**
- `https://www.googleapis.com/auth/spreadsheets` (read/write)

**Setup Steps:**
1. Nebula handles OAuth flow automatically
2. First run will prompt for Google account authorization
3. Tokens stored securely in Nebula vault
4. Auto-refresh on expiration

---

### Reading Data

```python
import requests

# Nebula provides authenticated session
sheet_id = 'YOUR_SHEET_ID_HERE'
range_name = 'Edge Candidates!A2:R100'

url = f'https://sheets.googleapis.com/v4/spreadsheets/{sheet_id}/values/{range_name}'
response = requests.get(url, headers={'Authorization': f'Bearer {access_token}'})
data = response.json()['values']

for row in data:
    market_title = row[1]
    edge = float(row[7].strip('%')) / 100
    # Process...
```

---

### Writing Data

```python
# Append new row
url = f'https://sheets.googleapis.com/v4/spreadsheets/{sheet_id}/values/{range_name}:append'
body = {
    'values': [[
        '2026-02-11',  # Date
        'Will NVDA beat earnings?',  # Market title
        'Stocks',  # Category
        0.72,  # Yes price
        0.28,  # No price
        0.72,  # Implied probability
        0.85,  # Research probability
        0.13,  # Edge
        'Medium',  # Edge size
        125000,  # Liquidity
        45000,  # Volume 24h
        '2026-02-15',  # Resolution date
        4,  # Days to close
        4,  # Risk score
        'BUY YES',  # Recommendation
        500,  # Position size
        'https://polymarket.com/...',  # URL
        'Historical 85% beat rate',  # Notes
        'Watching',  # Status
        '',  # Entry price (empty)
        '',  # Exit price (empty)
        ''   # P&L (empty)
    ]]
}

response = requests.post(
    url,
    json=body,
    headers={'Authorization': f'Bearer {access_token}'},
    params={'valueInputOption': 'USER_ENTERED'}
)
```

---

### Updating Existing Rows

```python
# Update specific range
range_name = 'Edge Candidates!D2:E2'  # Update Yes/No prices
url = f'https://sheets.googleapis.com/v4/spreadsheets/{sheet_id}/values/{range_name}'

body = {
    'values': [[
        0.68,  # New Yes price
        0.32   # New No price
    ]]
}

response = requests.put(
    url,
    json=body,
    headers={'Authorization': f'Bearer {access_token}'},
    params={'valueInputOption': 'USER_ENTERED'}
)
```

---

## Data Validation Rules

### Edge Candidates Tab

**Status Column (S):**
```
Dropdown: Watching, Entered, Closed
```

**Category Column (C):**
```
Dropdown: Sports, Politics, Crypto, Economics, Entertainment, Science
```

**Recommendation Column (O):**
```
Dropdown: BUY YES, BUY NO, PASS
```

### Trade Signal Log Tab

**Asset Type Column (C):**
```
Dropdown: Stock, Crypto, Polymarket, Options
```

**Signal Type Column (D):**
```
Dropdown: LONG, SHORT, BUY YES, BUY NO
```

**Conviction Column (I):**
```
Dropdown: Low, Medium, High
```

**Outcome Column (N):**
```
Dropdown: Win, Loss, Breakeven, Open
```

---

## Automation Workflows

### Workflow 1: Hourly Edge Detection

```
1. Script runs every 4 hours (trigger)
2. Fetch active Polymarket markets
3. Filter: Liquidity >$10K, resolves in >7 days
4. For each market:
   a. Check if already in sheet (by market ID)
   b. If new: Append row with research
   c. If existing: Update Yes/No prices
5. Sort by edge (descending)
6. Send alert if new edge >15% found
```

---

### Workflow 2: Daily Trade Log Update

```
1. User manually enters trade outcomes
2. Script runs nightly (or on-demand)
3. Read Trade Signal Log
4. Calculate:
   - Win rate
   - Average P&L
   - Sharpe ratio
5. Update Performance Dashboard
6. Generate weekly report
```

---

### Workflow 3: Position Monitoring

```
1. Trigger fires (hourly for active positions)
2. Read Edge Candidates where Status = "Entered"
3. For each position:
   a. Fetch current market price
   b. Calculate unrealized P&L
   c. Check if target or stop hit
   d. Update Status if closed
4. Send alerts for closed positions
```

---

## Manual Operations

### Adding a Market Manually

1. Open Edge Candidates tab
2. Insert new row
3. Fill required fields:
   - Date Added: `=TODAY()`
   - Market Title: Copy from Polymarket
   - Category: Select from dropdown
   - Yes/No Prices: Current prices
   - Implied Probability: `=Yes Price`
   - Research Probability: Your estimate
   - Edge: `=Research - Implied`
   - Status: "Watching"
4. Fill optional fields:
   - Notes: Research summary
   - Risk Score: 1-10 (lower = better)

---

### Recording a Trade

**When Entering:**
1. Update Status to "Entered"
2. Fill Entry Price column
3. Add notes about timing/reasoning

**When Exiting:**
1. Fill Exit Price column
2. Update Status to "Closed"
3. Realized P&L auto-calculates
4. Add post-trade review notes

---

### Cleaning Up Old Data

**Monthly Maintenance:**
1. Edge Candidates:
   - Archive closed positions (move to Archive tab)
   - Delete markets resolved >30 days ago
2. Trade Signal Log:
   - Keep all records (historical analysis)
   - Export to CSV for backup
3. Performance Dashboard:
   - Update charts to show last 90 days

---

## Troubleshooting

### Error: 401 Unauthorized
**Cause:** OAuth token expired  
**Solution:**
1. Re-run script from Nebula
2. Authorize when prompted
3. Test with simple read operation

### Error: 429 Too Many Requests
**Cause:** Rate limit exceeded (100 req/100s)  
**Solution:**
1. Implement exponential backoff
2. Batch operations (update multiple rows at once)
3. Cache data locally, update less frequently

### Error: Invalid Range
**Cause:** Range doesn't exist or typo  
**Solution:**
1. Verify tab name matches exactly (case-sensitive)
2. Check A1 notation (e.g., 'Sheet1!A1:B10')
3. Ensure sheet exists

### Data Not Updating
**Possible Causes:**
1. Script not running (check trigger status)
2. Sheet ID incorrect
3. Permissions issue (re-authorize)
4. API response changed (validate schema)

**Debug Steps:**
1. Run script manually with `debug=True`
2. Print API responses
3. Check Nebula execution logs
4. Test with Google Sheets API Explorer

---

## Advanced Features

### Conditional Formatting Scripts

```javascript
// Google Apps Script for custom formatting
function highlightEdges() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Edge Candidates');
  var range = sheet.getRange('H2:H100');  // Edge column
  var values = range.getValues();
  
  for (var i = 0; i < values.length; i++) {
    var edge = parseFloat(values[i][0]);
    if (edge > 0.15) {
      range.getCell(i+1, 1).setBackground('#00ff00');  // Green
    } else if (edge < -0.15) {
      range.getCell(i+1, 1).setBackground('#ff0000');  // Red
    }
  }
}
```

---

### Custom Functions

```javascript
// Calculate Kelly Criterion position size
function KELLY(edge, probability, bankroll) {
  var q = 1 - probability;
  var kelly = (probability / (1/edge)) - (q / edge);
  return bankroll * kelly * 0.5;  // Half-Kelly for safety
}

// Usage in sheet: =KELLY(H2, G2, 10000)
```

---

### Automated Email Alerts

```javascript
// Google Apps Script trigger (daily)
function checkForHighEdges() {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Edge Candidates');
  var data = sheet.getDataRange().getValues();
  
  var alerts = [];
  for (var i = 1; i < data.length; i++) {  // Skip header
    var edge = parseFloat(data[i][7]);  // Column H
    var riskScore = parseFloat(data[i][13]);  // Column N
    
    if (edge > 0.15 && riskScore < 5) {
      alerts.push(data[i][1]);  // Market title
    }
  }
  
  if (alerts.length > 0) {
    MailApp.sendEmail({
      to: 'dutchiono@gmail.com',
      subject: 'High-Conviction Edge Alert',
      body: 'New opportunities:\n' + alerts.join('\n')
    });
  }
}
```

---

## Related Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) - System design
- [SCRIPTS.md](SCRIPTS.md) - Script documentation
- [SETUP.md](SETUP.md) - Restoration guide
- [README.md](README.md) - Quick start

---

**Documentation Version:** 1.0  
**Last Updated:** 2026-02-11  
**Maintained By:** Nebula AI + dutchiono