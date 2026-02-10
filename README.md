## Yost Facilities — Ice Rink Dashboard

A lightweight, client-only dashboard for Yost Facilities staff to monitor ice rink operations. Data is pulled directly from published Google Sheets JSON endpoints; no backend or database is required.

### Features

- **Login gate**: Single shared password, stored only in the front-end config.
- **Main dashboard**: Header with branding, last refresh time, and logout button.
- **KPI cards**: Config-driven KPIs mapped to Google Sheet columns.
- **Recent logs table**: Shows the latest submissions for each form.
- **Multi-form support**: Switch between multiple Google Sheets sources.
- **Auto-refresh**: Periodic data refresh (configured in `config.js`).

### Getting Started

1. **Open the dashboard**
   - You can simply open `index.html` directly in a modern browser, or
   - Serve the folder with a small static server (recommended for fetch security):

```bash
cd /Users/raghu/YostDashboard/Yost-Facilities-Dashboard
python -m http.server 4173
```

Then visit `http://localhost:4173` in your browser.

2. **Login**
   - Default shared password is set in `config.js` under `loginPassword`.
   - Change this value to whatever you prefer.

### Configuring Google Sheets

**Yes, this is a template!** You can customize everything in `config.js`:
- Add/remove forms
- Change KPI names, units, and ranges
- Update column mappings
- Adjust refresh intervals

#### Step-by-Step: Setting Up Google Forms → Google Sheets → Dashboard

1. **Create a Google Form**
   - Go to [Google Forms](https://forms.google.com)
   - Create your form (e.g., "Daily Ice Rink Operations")
   - Add questions matching what you want to track (e.g., "Ice Temperature", "Attendance", "Maintenance Status")
   - Google will automatically create a response Sheet

2. **Get Your Sheet ID**
   - Open the response Sheet (click "Responses" tab → "Link to Sheets")
   - Look at the URL: `https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit`
   - Copy the `SHEET_ID_HERE` part

3. **Create a JSON Endpoint Using Google Apps Script**
   - In your Sheet, go to **Extensions → Apps Script**
   - Delete any default code and paste this:

```javascript
function doGet() {
  // Enable CORS for cross-origin requests
  const output = ContentService.createTextOutput();
  output.setMimeType(ContentService.MimeType.JSON);
  
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    const data = sheet.getDataRange().getValues();
    
    if (data.length === 0) {
      output.setContent(JSON.stringify([]));
      return output;
    }
    
    const headers = data[0];
    const rows = data.slice(1).map(row => {
      const obj = {};
      headers.forEach((header, i) => {
        obj[header] = row[i] || null;
      });
      return obj;
    });
    
    output.setContent(JSON.stringify(rows));
    return output;
  } catch (error) {
    output.setContent(JSON.stringify({ error: error.toString() }));
    return output;
  }
}
```

   - Click **Deploy → New deployment**
   - Choose type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone** (or "Anyone with Google account" if you want some protection)
   - Click **Deploy**
   - Copy the **Web app URL** (looks like `https://script.google.com/macros/s/.../exec`)

4. **Update `config.js`**
   - Open `config.js`
   - Find the form you want to configure (e.g., `"daily-ops"`)
   - Set `sheetJsonUrl` to your Apps Script URL from step 3
   - **Important**: Make sure `columns` keys match your Sheet's exact header names:
     ```javascript
     columns: {
       timestamp: "Timestamp",  // Must match Sheet header exactly
       iceTemperature: "Ice Temperature (°F)",
       // ... etc
     }
     ```
   - Customize `kpis` to match your columns and add/remove as needed

5. **Test It!**
   - Submit a test entry via your Google Form
   - Wait a few seconds for the form to process
   - Refresh your dashboard (or wait for auto-refresh)
   - You should see your real data!

**Note**: If `sheetJsonUrl` is left as `null`, the dashboard will show mock data so you can test the UI without setting up Sheets first.

### Troubleshooting "Failed to Fetch" Errors

If you see a "Failed to fetch" or network error:

1. **Check the URL format**
   - Production deployments use `/exec` at the end: `.../macros/s/.../exec`
   - Development URLs use `/dev`: `.../macros/s/.../dev`
   - Both should work, but `/exec` is recommended for production

2. **Verify deployment settings**
   - Go back to Apps Script → **Deploy → Manage deployments**
   - Click the pencil icon to edit
   - Make sure **Who has access** is set to **"Anyone"** (not "Anyone with Google account")
   - Click **Deploy** again if you changed it

3. **Test the URL directly**
   - Open your Apps Script URL in a browser
   - You should see JSON data (or an empty array `[]` if the sheet is empty)
   - If you see an error page, the script isn't deployed correctly

4. **Check browser console**
   - Open browser DevTools (F12) → Console tab
   - Look for detailed error messages that will help diagnose the issue

5. **Redeploy after script changes**
   - After updating your Apps Script code, you must **Deploy → Manage deployments → Edit → Deploy** again
   - The script won't update automatically

### Auto-Refresh

- The refresh interval is controlled by `APP_CONFIG.refreshIntervalMs` in `config.js`.
- Default is 3 minutes. Adjust as needed.

### Notes

- Everything runs client-side; there is no backend.
- The login is a simple password gate for convenience, not a secure authentication system.

