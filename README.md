# CargoEasy — Desktop App (Phase 1 prototype)

A real, installable desktop build of the CargoEasy freight-forwarding prototype, built with
Electron. Runs fully offline once installed (Google Fonts will silently fall back to system
fonts if you're offline — everything else works with no internet connection).

## What you need first

- **Node.js 18 or newer** — download from https://nodejs.org if you don't already have it.
  Installing Node also installs `npm`, which is all you need beyond that.

## 1. Install dependencies

Open a terminal in this folder (`cargoeasy-desktop`) and run:

```
npm install
```

This downloads Electron, React, and the build tools. It needs an internet connection and
can take a few minutes the first time.

## 2. Run it in development mode

```
npm start
```

This builds the app and opens it in a real window. Use this while you're testing changes.

## 3. Build a real installer

### Option A — build it yourself locally
On the OS you want an installer for, run:
```
npm run dist
```
This produces an installer for **whatever operating system you run it on**:
- On Windows → a `.exe` installer under `release/`
- On macOS → a `.dmg` under `release/`
- On Linux → a `.AppImage` under `release/`

### Option B — let GitHub build a Windows .exe for you (no Windows PC needed)
This project includes `.github/workflows/build.yml`, a GitHub Actions workflow that builds a
real Windows installer in the cloud on every push — you don't need Node, npm, or a Windows
machine yourself.

1. Create a new (free) repository on https://github.com and push this folder's contents to it:
   ```
   git init
   git add .
   git commit -m "CargoEasy desktop prototype"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<your-repo>.git
   git push -u origin main
   ```
2. On GitHub, open the **Actions** tab of your repository. You'll see a run called
   "Build CargoEasy Windows installer" start automatically (it also runs any time you push,
   or you can trigger it manually with the "Run workflow" button).
3. When it finishes (a few minutes), open that run and scroll to **Artifacts** —
   `CargoEasy-Windows-Installer` is a zip containing the real `.exe`. Download it, unzip it,
   and double-click the `.exe` to install CargoEasy on any Windows PC.

Either way, double-clicking the resulting installer installs CargoEasy like any other desktop
app — it'll appear in the Start Menu / Applications folder from then on.

## Where data is stored

- By default, CargoEasy stores its data in your OS's standard app-data folder.
- In the app, go to **Company Profile → Storage location → Choose folder…** to pick any
  folder instead — including a folder synced by Google Drive for Desktop, so your shipment
  data and exported documents sync to the cloud automatically.

## What's implemented in this Phase 1 build

- Role-based login (Viewer / Editor / Manager / Admin) — demo-only, no password
  verification yet (see "Next steps" below).
- Shipment intake form matching the shipping declaration layout (sender, receiver,
  shipment details, cargo contents, service options).
- CSV template download + import with preview.
- Company profile & logo branding.
- HBL document generation with print-to-PDF (use your OS print dialog → "Save as PDF").
- Local data persistence to a JSON store in your chosen folder.

## Suggested next steps (Phase 2+)

- Real authentication (hashed passwords, sessions) instead of the role-selector demo login.
- A proper database (SQLite via `better-sqlite3` is a natural fit for a desktop app like
  this) instead of the flat JSON store.
- The remaining reports: manifest, customs manifest grouping, waybill, delivery report,
  receivables/payables, tax invoices, P&L, job profitability.
- A genuine Google Drive API connection (OAuth) if you want live cloud sync rather than
  relying on the Drive desktop client mirroring a folder.
- Code-signing the installer (Windows/macOS both warn on unsigned apps) before distributing
  it outside your own machine.
# CargoEasy
