# Intelligent Data Analyzer — static build

This is a fully static, backend-free version of the Data Analyzer. Every bit of
analysis that used to run in `app.py`/`analyzer.py` on a FastAPI server now
runs client-side in the browser (`js/*.js`), so this folder can be hosted
anywhere that serves plain files — including GitHub Pages, for free, forever.

Nothing is uploaded anywhere. Files you pick are parsed and analyzed entirely
in your own browser tab; there's no server to send data to.

## Preview it locally before publishing

```bash
./serve.sh
```

Then open http://localhost:8020. (Just double-clicking `index.html` also
mostly works, but some browsers restrict local script loading over the
`file://` protocol — `serve.sh` avoids that entirely.)

## Publish to GitHub Pages

1. Create a new GitHub repository (public, so Pages is free) — either on
   github.com or with `gh repo create`.
2. From this folder:
   ```bash
   git init
   git add .
   git commit -m "Static data analyzer"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo>.git
   git push -u origin main
   ```
3. On GitHub: repo → **Settings → Pages** → under "Build and deployment",
   set **Source** to "Deploy from a branch", branch `main`, folder `/ (root)`.
   Save.
4. Wait ~1 minute, then your app is live at
   `https://<your-username>.github.io/<your-repo>/`.

## Updating it later

Edit the files, commit, and push — GitHub Pages redeploys automatically on
every push to `main`. No server to restart, nothing to redeploy manually.

## What changed vs. the FastAPI version

- `analyzer.py` → `js/core.js`, `js/schema.js`, `js/forecast.js`,
  `js/goals.js`, `js/insights.js`, `js/run-analysis.js` (the analysis engine,
  ported function-for-function).
- `app.py`'s session/`/api/*` endpoints → `js/session.js` (an in-memory
  browser-side equivalent — `FILES`/`ACTIVE_FILE_ID`/`COMPARE_ANALYZER`
  instead of server-side session dicts).
- `index.html` — same UI and chart-rendering code as before; only the ~10
  `fetch('/api/...')` call sites were swapped for direct local function calls.
- CSV parsing is hand-rolled; Excel parsing uses the SheetJS (`xlsx.js`)
  library already loaded on the page.

### Known limitations vs. the Python version

- Date parsing is a best-effort port (ISO / `MM/DD/YYYY` / native `Date`
  fallback), not a full port of Python's `dateutil` — very unusual date
  formats may not parse.
- State lives only in the current tab (by design, since there's no server) —
  refreshing the page clears loaded files, same as closing any browser tab
  with unsaved in-memory state.
- Very large files (hundreds of thousands of rows) will run slower here than
  on a pandas backend, since the aggregation logic isn't vectorized in C —
  fine for typical exports, worth knowing for huge ones.
