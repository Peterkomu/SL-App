# Dagstaat App (no-Node version)

A single-file web form that replaces the paper dagstaat. Drivers open a
link, fill it in on their phone, and every submission lands in your
Google Sheet.

**No Node.js, no build step, no server to manage.** Just:

- `index.html` — the whole app (one file) → hosted on **Vercel**, code in **GitHub**
- `apps-script-Code.gs` — a small script that lives inside your Google
  Sheet and writes the incoming data into it

The form talks to the sheet through a **Google Apps Script Web App URL**.

---

## Overview of the 3 pieces

```
  Driver's phone                Vercel                 Google Sheet
 ┌──────────────┐   opens   ┌──────────────┐        ┌──────────────┐
 │  the link    │ ────────► │  index.html  │        │   Shifts tab │
 │  fills form  │           │  (your form) │        │   Stops  tab │
 └──────────────┘           └──────┬───────┘        └──────▲───────┘
                                   │  POST data            │
                                   ▼                       │
                          ┌────────────────────┐  writes   │
                          │ Google Apps Script │ ──────────┘
                          │   (Web App URL)    │
                          └────────────────────┘
```

---

## STEP 1 — Set up the Google Sheet + Apps Script

1. Create a new Google Sheet (call it e.g. "Dagstaat Data"). You don't
   need to add tabs or headers manually — the script creates the
   `Shifts` and `Stops` tabs (with headers) automatically on the first
   submission.
2. In that sheet, go to **Extensions → Apps Script**.
3. Delete whatever code is in the editor, then open the file
   `apps-script-Code.gs` from this project, copy ALL of it, and paste it
   in. Click the **save** (disk) icon.
4. Click **Deploy → New deployment**.
5. Click the gear icon next to "Select type" and choose **Web app**.
6. Set:
   - **Description**: anything (e.g. "Dagstaat endpoint")
   - **Execute as**: **Me**
   - **Who has access**: **Anyone**
7. Click **Deploy**. Google will ask you to authorize — approve it (you
   may see a "Google hasn't verified this app" warning; click *Advanced
   → Go to (your project)* → *Allow*. This is normal for your own
   script.)
8. Copy the **Web app URL** it shows you. It looks like:
   `https://script.google.com/macros/s/AKfyc..../exec`

   Keep this URL — you need it in the next step.

---

## STEP 2 — Put your URL into index.html

1. Open `index.html` in any text editor (Notepad works).
2. Near the top of the `<script>` section, find this line:
   ```js
   var SCRIPT_URL = "PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE";
   ```
3. Replace the placeholder with your real Web App URL from Step 1:
   ```js
   var SCRIPT_URL = "https://script.google.com/macros/s/AKfyc..../exec";
   ```
4. A few lines below, edit the driver list if you like:
   ```js
   var DRIVERS = ["Ionut", "Erik van der Wal", "Kay Fonteijn"];
   ```
5. Save the file.

That's the only editing you ever need to do in the HTML.

---

## STEP 3 — Put the code on GitHub

You can do this entirely in the browser — no git command line needed:

1. Go to [github.com](https://github.com) and create a new repository
   (e.g. `dagstaat-app`). Leave it empty (no README).
2. On the new repo page, click **uploading an existing file**.
3. Drag in `index.html`, `vercel.json`, and (optional) `README.md`.
   You do NOT need to upload `apps-script-Code.gs` — that lives in your
   Google Sheet, not on the website. (Uploading it does no harm though.)
4. Click **Commit changes**.

---

## STEP 4 — Deploy on Vercel

1. Go to [vercel.com](https://vercel.com) and sign in (you can sign in
   with your GitHub account).
2. Click **Add New → Project**.
3. Import the `dagstaat-app` repo you just created.
4. Vercel sees it's a plain static site — you don't need to change any
   settings. Just click **Deploy**.
5. After a few seconds you get a link like
   `https://dagstaat-app.vercel.app`.

That link **is your app.** Every time you change `index.html` on GitHub,
Vercel redeploys automatically.

---

## STEP 5 — Send the link to drivers

Share the Vercel URL by WhatsApp / text / however you reach them. They
open it, pick their name, fill in the shift and stops, and tap **Dagstaat
verzenden**. Within a second or two the data appears as new rows in your
Google Sheet:

- one row per shift in the **Shifts** tab (with auto-calculated bruto/
  netto uren and KM totaal)
- one row per stop in the **Stops** tab, linked to its shift by `ShiftId`

---

## Testing it works

Before handing the link to drivers, open it yourself, fill in a test
entry (at minimum a driver name + date), and submit. Check that a new
row appears in the sheet. If it doesn't:

- Double-check `SCRIPT_URL` in `index.html` matches the deployed Web App
  URL exactly (ends in `/exec`, not `/dev`).
- Make sure the Apps Script deployment's "Who has access" is **Anyone**.
- If you changed the Apps Script code after deploying, you must do
  **Deploy → Manage deployments → Edit → Version: New version** for the
  change to take effect (or the URL keeps running the old code).

---

## Downloading / sharing a PDF

Drivers can tap **Download PDF** — either on the main form or on the
confirmation screen after submitting — to get a PDF laid out like the
paper dagstaat (header fields, Diensttijd/Kilometers boxes, the stop
table, and Bijzonderheden). On a phone this opens the normal share
sheet, so they can send it by WhatsApp, email, etc.

The PDF is generated entirely on the phone (via the jsPDF library loaded
from a CDN) — nothing extra to set up. The only requirement is an
internet connection so the library can load, which the driver already
needs in order to submit to the sheet.

The filename is automatically `Dagstaat_<naam>_<datum>.pdf`.

## Changing the driver list later

Edit the `DRIVERS = [...]` line in `index.html`, commit the change on
GitHub, and Vercel redeploys automatically. No other steps.

## Editing or correcting data

The app only *adds* rows. To fix a mistake, edit the Google Sheet
directly.
