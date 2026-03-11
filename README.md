# 🚗 Shadows Car Wash — Membership Portal

A free, lightweight membership tracking website for **Shadows Car Wash**. Staff can register customers, record washes, and view history. Customers can check their wash count and loyalty tier instantly.

**Live Site:** [https://naveenkumarvedarajan.github.io/Shadowscarwash](https://naveenkumarvedarajan.github.io/Shadowscarwash)

---

## Features

- 🔍 **Customer lookup** — search by vehicle number to see wash count, tier, and full history
- 📅 **Wash history with date & time** — every wash is recorded automatically
- 🏅 **Loyalty tiers** — Bronze → Silver → Gold → Platinum based on wash count
- 📊 **Progress bar** — shows how close the customer is to the next tier
- ➕ **Register new customers** — staff can add name, vehicle number, and phone
- ✦ **Record washes** — one click saves a wash directly to Google Sheets
- ⬇ **Excel export** — download all customer data as a `.xlsx` file
- ☁️ **Google Sheets backend** — all data stored live in Google Sheets

---

## Files

| File | Description |
|------|-------------|
| `index.html` | Main website (UI + layout) |
| `app.js` | All JavaScript logic + Google Sheets connection |

---

## How It Works

1. **Customer** opens the site → enters vehicle number → sees wash count, tier, and history
2. **Staff** opens the Staff Panel → records a wash or registers a new customer → data saves to Google Sheets instantly
3. **Admin** can export all data as Excel from the Staff Panel

---

## Setup (Google Sheets Backend)

### Step 1 — Create Google Sheet
1. Go to [sheets.google.com](https://sheets.google.com) and create a new sheet named **Shadows Car Wash**
2. Add these headers in Row 1:

| A | B | C | D | E | F |
|---|---|---|---|---|---|
| Vehicle | Name | Phone | Wash Count | Last Wash Date | Wash History |

### Step 2 — Create Apps Script
1. In the sheet, click **Extensions → Apps Script**
2. Delete all existing code and paste the following:

```javascript
function doGet(e) {
  var callback = e.parameter.callback;
  var action = e.parameter.action;
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var rows = sheet.getDataRange().getValues();
  var result = {};

  if (action === "lookup") {
    var vehicle = e.parameter.vehicle;
    var found = false;
    for (var i = 1; i < rows.length; i++) {
      if (String(rows[i][0]).toUpperCase() === String(vehicle).toUpperCase()) {
        result = { found:true, vehicle:rows[i][0], name:rows[i][1],
          phone:rows[i][2], count:rows[i][3], lastDate:rows[i][4], history:rows[i][5] };
        found = true; break;
      }
    }
    if (!found) result = { found: false };
  }

  if (action === "addWash") {
    var vehicle = e.parameter.vehicle;
    var name = e.parameter.name || "";
    var phone = e.parameter.phone || "";
    var found = false;
    for (var i = 1; i < rows.length; i++) {
      if (String(rows[i][0]).toUpperCase() === String(vehicle).toUpperCase()) {
        var count = (parseInt(rows[i][3]) || 0) + 1;
        var date = new Date().toLocaleString("en-IN");
        var history = rows[i][5] ? rows[i][5] + "\n" + date : date;
        sheet.getRange(i+1,4).setValue(count);
        sheet.getRange(i+1,5).setValue(date);
        sheet.getRange(i+1,6).setValue(history);
        result = { success:true, count:count, history:history };
        found = true; break;
      }
    }
    if (!found) {
      var date = new Date().toLocaleString("en-IN");
      sheet.appendRow([vehicle, name, phone, 1, date, date]);
      result = { success:true, count:1, history:date };
    }
  }

  if (action === "export") {
    result = { rows: rows.slice(1) };
  }

  return ContentService.createTextOutput(callback + "(" + JSON.stringify(result) + ")")
    .setMimeType(ContentService.MimeType.JAVASCRIPT);
}
```

3. Click **Save (💾)**

### Step 3 — Deploy
1. Click **Deploy → New Deployment**
2. Click the ⚙️ gear → select **Web App**
3. Set **Execute as: Me** and **Who has access: Anyone**
4. Click **Deploy** → Authorize → Allow
5. Copy the Web App URL and update it in `app.js`

---

## Loyalty Tiers

| Tier | Washes Required |
|------|----------------|
| 🥉 Bronze | 0–4 washes |
| 🥈 Silver | 5–9 washes |
| 🥇 Gold | 10–19 washes |
| 💎 Platinum | 20+ washes |

---

## Tech Stack

- Plain HTML, CSS, JavaScript (no frameworks)
- Google Apps Script (backend / database)
- Google Sheets (data storage)
- SheetJS / xlsx.js (Excel export)
- Hosted on GitHub Pages (free)

---

## License

This project is private and built for **Shadows Car Wash — Premium Detailing**.
