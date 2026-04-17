# Abbott KH · Total Return Dashboard

Live dashboard hosted on **Vercel** + **GitHub**, with data served from a **Google Apps Script API** that reads directly from Google Sheets.

## Architecture

```
  Google Sheet  →  Apps Script (API)  →  Vercel (HTML)  →  Browser
  (data source)    (cached JSON)        (static file)     (interactive)
```

- **Google Sheet** — source of truth, updated whenever your team adds rows
- **Apps Script** — fetches sheet data, cleans it, caches 6 hours, serves as JSON API
- **Vercel** — hosts the HTML dashboard, deploys instantly from GitHub
- **GitHub** — version control + triggers Vercel auto-deploy on every commit

---

## Part 1 · Deploy the Apps Script API (~5 min)

This step turns your Google Sheet into a JSON API that the dashboard can call.

### 1.1 · Open Apps Script

- Open your Google Sheet
- Top menu: **Extensions → Apps Script**

### 1.2 · Paste the API code

- Delete whatever is in `Code.gs`
- Open `apps-script-api/Code.gs` from this project, copy ALL of it, paste into Apps Script
- Press **Ctrl + S** to save
- Rename the project (top-left) to `Returns API`

### 1.3 · Test it

- In the function dropdown at top, pick `TEST_api`
- Click **Run**
- Authorize when asked (click Advanced → Go to project → Allow — it's your own script)
- Check the execution log — you should see:
  ```
  Records: 9712 | Months: 16 | Brands: 7
  ```

### 1.4 · Deploy as Web App

- Top-right: **Deploy → New deployment**
- Click the gear icon → pick **Web app**
- Set:
  - **Execute as**: Me
  - **Who has access**: **Anyone** ← ⚠️ **This is important** — Vercel needs to be able to fetch from it without a login. This makes the URL publicly fetchable, but the URL is unguessable so it's practically secure.
- Click **Deploy**
- **Copy the URL** — it looks like:
  ```
  https://script.google.com/macros/s/AKfycbx.../exec
  ```
  Save this — you'll need it in Part 2.

### 1.5 · (Optional) Install daily pre-warm trigger

- In the function dropdown, pick `INSTALL_dailyTrigger`
- Click **Run**
- This makes the dashboard instant every morning instead of taking 15s for the first visitor.

---

## Part 2 · Push to GitHub (~3 min)

### 2.1 · Create a new GitHub repo

- Go to <https://github.com/new>
- Repo name: `returns-dashboard` (or whatever you like)
- Keep it **Private** (recommended) — the dashboard is public via Vercel but the source doesn't need to be
- Click **Create repository**

### 2.2 · Upload the files

Easiest way via the web interface:

- On the new repo page, click **uploading an existing file**
- Drag and drop these files from this project:
  - `index.html`
  - `vercel.json`
  - `.gitignore`
  - `README.md` (this file)
- **Do NOT upload the `apps-script-api/` folder** — that's for Apps Script, not Vercel
- Click **Commit changes**

**OR** via command line:

```bash
cd path/to/this/project
git init
git add index.html vercel.json .gitignore README.md
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/returns-dashboard.git
git push -u origin main
```

### 2.3 · Edit index.html to point to your API

Before or after uploading, edit `index.html` and find this line (near line 317):

```javascript
const API_URL = 'YOUR_APPS_SCRIPT_WEB_APP_URL_HERE';
```

Replace with the URL you copied in step 1.4:

```javascript
const API_URL = 'https://script.google.com/macros/s/AKfycbx.../exec';
```

Commit and push this change. (You can also edit it directly on GitHub — just click the pencil icon on the file.)

---

## Part 3 · Deploy to Vercel (~2 min)

### 3.1 · Sign up / log in

- Go to <https://vercel.com>
- Sign up with **your GitHub account** (easiest)

### 3.2 · Import the repo

- On Vercel dashboard: **Add New → Project**
- Find your `returns-dashboard` repo → **Import**
- Framework Preset: **Other** (it's a static site)
- Leave everything else default
- Click **Deploy**

Vercel builds it in ~10 seconds and gives you a URL like:

```
https://returns-dashboard.vercel.app
```

🎉 That's your live dashboard URL. It's permanent, served over HTTPS, fast (Vercel's global CDN), and updates automatically whenever you push to GitHub.

### 3.3 · (Optional) Custom domain

- In Vercel project → **Settings → Domains**
- Add your own domain (e.g., `dashboard.yourcompany.com`)
- Follow the DNS instructions Vercel shows

---

## How updates work

| You do this | Result |
|---|---|
| Add rows in Google Sheet | Dashboard shows new data within 6 hours (cache expires) OR instantly when someone clicks **Refresh Data** |
| Push code change to GitHub | Vercel auto-deploys the new version in ~15 seconds |
| Edit `index.html` directly on GitHub | Same — auto-deploys |

---

## Troubleshooting

### Dashboard shows "Could not load data"

- Check that `API_URL` in `index.html` is correct
- Make sure the Apps Script deployment has **Who has access: Anyone**
- Try opening the API URL directly in a browser — you should see raw JSON

### Data is stale / not showing latest rows

- Click the **Refresh Data** button in the top-right corner of the dashboard
- OR visit `YOUR_API_URL?refresh=1` once in your browser to force cache clear
- OR wait up to 6 hours for the automatic cache expiry

### "Unauthorized" or "redirect" error

- The Apps Script needs to be deployed with **Who has access: Anyone** (not "Anyone with Google account")
- Re-deploy: **Manage deployments → ✏ → Who has access: Anyone → Deploy**

### I need to change something later

- Edit the file in GitHub (pencil icon, commit) → Vercel auto-deploys
- For Apps Script changes: edit in the Apps Script editor → **Deploy → Manage deployments → ✏ → New version → Deploy** (keeps same URL)

---

## Security

- **The dashboard URL** (Vercel) is public. Anyone with the link can view.
- **The API URL** (Apps Script) is also public but unguessable. It only returns aggregated/cleaned data, never the raw sheet rows with full details.
- **Your Google Sheet** is not shared publicly — only the Apps Script can read it because it runs as you.

If you need authentication (e.g., restrict to company email domains), that's a bigger upgrade — let me know and I'll add it.

---

## Files in this project

```
returns-dashboard/
├── index.html          → dashboard UI (goes on Vercel)
├── vercel.json         → Vercel config (goes on Vercel)
├── .gitignore          → ignored files
├── README.md           → this file
└── apps-script-api/
    └── Code.gs         → paste into Google Apps Script (NOT on Vercel)
```
