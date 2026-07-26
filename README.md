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

- **Users and shipment records** live in a real SQLite database
  (`cargoeasy.db`) in your OS's standard app-data folder — this location never
  changes, so your login credentials and shipment data are never at risk from
  a folder change.
- **Exported documents** (CSV backups, future file exports) save to whatever
  folder you pick in **Company Profile → Storage location → Choose folder…**
  — point this at a folder synced by Google Drive for Desktop to get
  documents into the cloud automatically.

## Why sql.js instead of better-sqlite3

An earlier version of this project used `better-sqlite3`, which needs a C++
compiler (Visual Studio Build Tools on Windows) to build during `npm install`.
That's a common source of setup pain — especially with very new Visual
Studio releases that `node-gyp` doesn't recognize yet. `sql.js` is SQLite
compiled to WebAssembly: it's pure JavaScript + a `.wasm` file, so there is
**nothing to compile, on any platform, ever**. The trade-off is that sql.js
keeps the whole database in memory and we explicitly write it to disk after
every change (see `persist()` in `main.js`) — completely fine at this
prototype's scale, but worth knowing if the dataset grows very large.

## First run

On first launch, CargoEasy creates a default Admin account and shows the
generated email + temporary password **once**, right on the login screen.
Sign in with those, then go to **Users & Roles** to change that password and
add real accounts for your team (Viewer / Editor / Manager / Admin).

## What's implemented (Phase 1 + Phase 2)

- **Real authentication** — email + password, hashed with Node's built-in
  scrypt (no plaintext passwords anywhere), backed by a SQLite `users` table.
  Admins can add/remove users and everyone can change their own password.
- Real database — shipments, users, and company profile all persist in
  SQLite (via `sql.js`, a WebAssembly build — no native compilation needed
  on any platform) instead of a flat JSON blob.
- Shipment intake form matching the shipping declaration layout (sender,
  receiver, shipment details, cargo contents, service options).
- CSV template download + import with preview.
- Company profile & logo branding, including customs/CHA details.
- All 19 reports from the original spec — manifest, customs manifest
  (grouped HBLs), waybill, delivery, customer authorisation, inventory
  (loaded/unloaded), receivables, payables, additional duty, overweight
  payment, delivery payment, duty payment, tax invoices (sender + receiver),
  despatch note, P&L, job profitability, customs pending, warehouse
  discrepancy — all print-to-PDF ready.
- **Manual Paid/Unpaid toggle** on the Receivables report — click a
  shipment's status badge (Editor role and above) to flip it; this is now a
  real per-shipment flag in the database, not a guess based on freight terms.

## Honest caveats — still worth knowing

- Financial figures across the reports (freight rates, duty %, overweight
  surcharge, estimated cost) are placeholder rate assumptions, clearly
  labelled in the Reports Center. Replace with your real tariffs before using
  these for actual invoicing or accounting.
- There's no password-reset-by-email flow (no mail server in a desktop app
  like this) — an Admin resets a locked-out user by deleting and re-creating
  their account, or you extend `users:changePassword` with an admin-override
  path.
- IPC handlers trust whatever role the renderer sends for now — fine for a
  single-user desktop app, but if you ever expose this over a network, add
  server-side role checks in `main.js`, not just the UI hiding buttons.

## Suggested next steps (Phase 3+)

- A genuine Google Drive API connection (OAuth) if you want live cloud sync
  rather than relying on the Drive desktop client mirroring a folder.
- Code-signing the installer (Windows/macOS both warn on unsigned apps)
  before distributing it outside your own machine.
- Customer-facing tracking portal, email notifications, and audit logging of
  who changed what.
