# Deployment & Data Connection Guide

## Deployment options

### Option 1: GitHub Pages (recommended)

1. Go to your GitHub repo → **Settings** → **Pages**
2. Under **Source**, select **Deploy from a branch**
3. Set **Branch** to `main`, folder to `/ (root)`
4. Click **Save**
5. Your tool will be live at `https://[your-org].github.io/[repo-name]/` within ~60 seconds
6. Every push to `main` triggers an automatic re-deploy — no manual steps needed

> Note: GitHub Pages on private repos requires GitHub Pro or Teams. If your org is on the free tier, use SharePoint as the primary host and keep this repo for version control only.

### Option 2: SharePoint embed

1. Upload `index.html` **and** `keywords.json` to the same SharePoint document library
2. Navigate to the SharePoint page where you want the tool to appear
3. Click **Edit page** → **Add web part** → **Embed**
4. Paste the direct URL to `index.html` in the embed URL field
5. Set the web part height to **900px**
6. Publish the page

> Note: SharePoint may flag the embed with a security warning if the file isn't in a "trusted" location. If this happens, contact IT to whitelist the document library domain, or request an iframe exception.

### Option 3: Local / offline

1. Download `index.html` and `keywords.json` to the **same local folder**
2. Open `index.html` in Chrome, Edge, or Firefox
3. No internet connection required after download

> Note: The tool loads Tabler Icons from a CDN (`cdn.jsdelivr.net`). If you need fully offline use, download the icon font files and update the `<link>` tag in index.html to point to the local copy.

---

## Connecting live keyword data

### Current architecture

The tool fetches `keywords.json` from the same directory on load. If the file is not found (or a CORS error occurs), it silently falls back to built-in seed terms. To update keywords: edit `keywords.json` and push/upload. No changes to `index.html` are needed.

The config line in `index.html` that controls the data source:

```javascript
// KEYWORDS DATA SOURCE — change this URL to connect a live API endpoint
const KEYWORDS_URL = './keywords.json';
```

---

### Option A: Google Apps Script (fastest path to live BigQuery data)

1. **In BigQuery**, create a scheduled query that exports keyword data to a Google Sheet. Suggested query:

```sql
SELECT
  search_term,
  market_category,
  SUM(impressions) AS vol
FROM `your-project.amazon_ads.search_terms`
WHERE date >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY 1, 2
ORDER BY vol DESC
LIMIT 500
```

2. In the destination Google Sheet, go to **Extensions → Apps Script**

3. Create a `doGet()` function that returns the sheet data as JSON with CORS headers:

```javascript
function doGet() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const rows = data.slice(1).map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  });
  return ContentService
    .createTextOutput(JSON.stringify({ terms: rows }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. **Deploy as a Web App**: Execute as **Me**, Who has access: **Anyone in [your org]**

5. In `index.html`, find the config line and replace the URL:

```javascript
const KEYWORDS_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
```

6. Done. The tool now fetches live data from BigQuery via Sheets on every load.

---

### Option B: Google Cloud Run (production-grade)

For when IT is ready to stand up a proper backend.

1. Create a Cloud Run service in your existing GCP project
2. Use a service account with **BigQuery Data Viewer** and **BigQuery Job User** roles
3. The service accepts `GET /keywords` and returns JSON matching the `keywords.json` schema
4. Protect the endpoint with **Identity-Aware Proxy (IAP)** for internal-only access
5. Replace `KEYWORDS_URL` in `index.html` with the Cloud Run endpoint URL

This approach gives you: proper auth, caching, rate limiting, and full separation between the data pipeline and the front-end tool.

---

### Option C: Helium10 direct export (manual, no backend needed)

1. Export keyword data from Helium10 as CSV (Cerebro or Magnet reports)
2. Use a conversion script (`scripts/helium10-to-json.py` — add this when ready) to reformat to `keywords.json` schema
3. Commit the updated `keywords.json` and push to main
4. Recommended cadence: **monthly**, or before major catalog pushes

---

## Search volume display

When `vol` fields in `keywords.json` are populated with actual numbers, the Term Bank tab will:

- Sort terms by volume descending within each market
- Display the volume number next to each term chip
- Show a "live" or "seeded" data source badge in the Term Bank header

To populate volume: add integer search volume values to the `vol` field in `keywords.json`, or connect a live data source per Option A or B above.

---

## Access control

| Scenario | Recommendation |
|---|---|
| Internal team, GitHub Teams/Pro org | Private repo + GitHub Pages |
| Internal team, GitHub Free org | Private repo (version control only) + SharePoint embed |
| Broader internal distribution | SharePoint embed with standard SharePoint permissions |
| External sharing needed | Do not share without approval — contact marketing leadership |
