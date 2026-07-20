# Kasatria People Table — Setup Guide

A single-file web app (`index.html`) that adapts the Three.js
[CSS3D Periodic Table](https://threejs.org/examples/#css3d_periodictable) demo
to visualize the assignment's people dataset in 3D, gated behind Google Sign-In.

**It works out of the box in demo mode** — open `index.html` in a browser and
click "Continue in demo mode" to see all 200 records from `Data_Template.csv`
rendered as Table / Sphere / Double-Helix / Grid. The steps below turn on the
real Google Sign-In + live Google Sheet requirements from the assignment.

---

## 1. Create the Google Sheet

1. Go to [Google Sheets](https://sheets.google.com) → **Blank spreadsheet**.
2. `File > Import > Upload` and select `Data_Template.csv`. Import as
   "Replace current sheet."
3. Keep the header row exactly as-is: `Name, Photo, Age, Country, Interest, Net Worth`.
4. Click **Share** (top right) and add `lisa@kasatria.com` as a viewer (per
   the assignment instructions).
5. Publish it so the webpage can read it live:
   `File > Share > Publish to web` → select the sheet/tab → format **CSV** → **Publish**.
6. Copy the generated URL — it looks like:
   `https://docs.google.com/spreadsheets/d/e/2PACX-XXXXXXXX/pub?output=csv`

## 2. Create the OAuth Client ID (Google Cloud)

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → create
   a new project (or reuse one).
2. **APIs & Services > OAuth consent screen** → set it up (External is fine
   for testing) → add your own Google account as a test user.
3. **APIs & Services > Credentials > Create Credentials > OAuth client ID**.
   - Application type: **Web application**
   - **Authorized JavaScript origins**: add the URL you'll host the page at,
     e.g. `https://yourname.github.io` or `http://localhost:8080` while testing
   - Leave "Authorized redirect URIs" blank — Google Identity Services'
     button flow doesn't need one.
4. Copy the generated **Client ID** (ends in `.apps.googleusercontent.com`).

## 3. Configure `index.html`

Open `index.html` and edit the `CONFIG` object near the top of the `<script>` block:

```js
const CONFIG = {
  GOOGLE_CLIENT_ID: "1234567890-abc123.apps.googleusercontent.com",
  SHEET_CSV_URL: "https://docs.google.com/spreadsheets/d/e/2PACX-XXXX/pub?output=csv"
};
```

That's it — no build step, no dependencies to install. All libraries
(Three.js, CSS3DRenderer, OrbitControls, Tween.js, PapaParse, Google
Identity Services) load from CDNs.

## 4. Host the page

Any static host works, e.g.:

- **GitHub Pages**: push `index.html` to a repo, enable Pages on the `main`
  branch. Add the resulting URL as an Authorized JavaScript origin (step 2).
- **Netlify Drop**: drag the folder onto [app.netlify.com/drop](https://app.netlify.com/drop).
- **Firebase Hosting**: `firebase init hosting` → `firebase deploy`.

Whatever URL you end up with, make sure it's added under **Authorized
JavaScript origins** in the OAuth Client ID settings, or Google Sign-In will
reject it.

## 5. Verify against the assignment checklist

| # | Requirement | Where it's handled |
|---|---|---|
| 1 | Import CSV into Google Sheet, share with lisa@kasatria.com | Step 1 above (manual, on your Google account) |
| 2 | Webpage gated with Google login | `#login-screen`, Google Identity Services button |
| 3 | Retrieves data from the Google Sheet | `loadData()` fetches `CONFIG.SHEET_CSV_URL` and parses it with PapaParse |
| 4 | Tiles show the given data (photo, name, age, country, interest, net worth) instead of chemical elements | `buildObjects()` |
| 5 | Tile background colored by Net Worth (Red < $100K, Orange $100K–$200K, Green > $200K) | `.nw-red / .nw-orange / .nw-green` classes |
| 6 | Four layouts: Table, Sphere, Helix, Grid | `#menu` buttons + `buildTableTargets/buildSphereTargets/buildDoubleHelixTargets/buildGridTargets` |
| 7 | Table arranged 20 × 10 | `buildTableTargets()` (`COLS = 20, ROWS = 10`) |
| 8 | Double helix, not single | `buildDoubleHelixTargets()` interleaves two strands 180° apart |
| 9 | Grid arranged 5 × 4 × 10 | `buildGridTargets()` (`GX=5, GY=4, GZ=10`) |
| 10 | Send the URL | once hosted, share that link |

## Notes / things worth mentioning in your submission

- Photos are loaded directly from the `static.kasatria.com` URLs in the
  sheet; a tile falls back to initials if an image 404s.
- The 200 rows in the CSV divide evenly into the 20×10 table (200) and the
  5×4×10 grid (200), so no padding/truncation logic was needed — if the
  sheet ever has a different row count, the code degrades gracefully (extra
  tiles just won't have a Grid/Table slot to animate into).
- Currency values like `$251,260.80` are parsed with
  `parseCurrency()` before being compared against the $100K/$200K thresholds.
- If the Google Sheet fetch fails (not published, wrong URL, offline), the
  app automatically falls back to the embedded copy of the CSV so the demo
  never breaks — a message at the bottom-right ("Loaded N rows from Google
  Sheet" vs "Using embedded demo data") tells you which source is live.
