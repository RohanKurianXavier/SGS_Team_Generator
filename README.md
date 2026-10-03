# SGS Sports Randomizer

Draws the SGS Sports Event registrants into four balanced teams — Panthers,
Titans, Stormers and Falcons — with equal representation in every age group,
then lets you adjust the result by hand and export it.

**Live app:** https://USERNAME.github.io/sgs-sports-randomizer/
_(replace `USERNAME` once Pages is switched on)_

---

## Publishing it

The whole app is one file. There is nothing to build and no server to run.

1. On GitHub, click **New repository**. Name it `sgs-sports-randomizer`,
   set it to **Public**, and create it.
2. On the empty repo page choose **uploading an existing file**, then drag in
   `index.html` and `README.md` from this folder. Commit.
3. Go to **Settings → Pages**. Under *Build and deployment* set
   **Source: Deploy from a branch**, **Branch: `main`**, folder **`/ (root)`**.
   Save.
4. Wait about a minute, then open
   `https://USERNAME.github.io/sgs-sports-randomizer/`.

Anyone with that address can now use every feature — no account, no login,
works on a phone. To publish a change, upload a new `index.html` over the old
one; the address stays the same.

## Sharing a specific draw

Each visitor's browser keeps its own copy of the teams, so a bare link gives
the person who opens it a fresh draw of their own. To send *your* teams:

1. Press **Share draw**.
2. Paste your published address into the first box, once — it is remembered.
3. Copy the link and send it.

The teams ride inside the link itself (about 400 characters), so whoever
opens it lands on exactly your draw and can keep working from there.

## Making everyone's changes land in the sheet (optional)

By default each browser keeps its own copy. To make one Google Sheet the
single source of truth — the page loads the current teams when it opens, and
a **Save to sheet** button publishes changes back — deploy the script in
`apps-script/Code.gs`:

1. Open the Google Sheet you want to hold the teams (the form's own response
   sheet is the natural one). **Extensions → Apps Script**.
2. Delete whatever is in `Code.gs` and paste in the contents of
   `apps-script/Code.gs` from this repo. Save.
3. **Deploy → New deployment → Web app.** Description: anything.
   **Execute as: Me.** **Who has access: Anyone.** Deploy, and authorise it
   when Google asks (you will have to click through the "unverified app"
   warning — it is your own script).
4. Copy the **Web app URL**. It ends in `/exec`.
5. Open `index.html`, find `var SHEET_API = "";` near the top of the script
   block, and paste the URL between the quotes. Upload the file to GitHub
   again.

From then on the page is live for everyone:

- Opening it pulls the current teams from the sheet.
- **Every change saves itself** about a second and a half after you stop —
  no button to remember. That covers adding, moving, swapping, deleting,
  drawing, importing, restoring the original sheet, toggling gender balance,
  and **Undo** (an undo is a change like any other; if it did not publish, the
  next poll would quietly pull the undone version back). A dot in the status
  bar shows saving / saved / offline.
- Other people's changes arrive on their own. An open tab checks the sheet
  every 30 seconds (only while it is the visible tab) and takes anything newer.
- **Save now** is there as a manual nudge if you want one.

If the sheet cannot be reached, your change is kept in the browser and the
status bar says so; it goes out on the next successful save.

### When two people edit at once

Each save carries the revision it was based on. If the sheet has moved on
since, the script refuses the write and hands back the newer draw. The page
then takes that newer draw, **replays your own changes on top of it**, and
saves again — so both people's work survives rather than one overwriting the
other. Tested with two browsers adding players simultaneously: both landed.

The one case that cannot be merged is a wholesale roster replacement (Import
roster or Restore original sheet) racing someone else's edit. There the page
keeps theirs, discards yours and says so, rather than guessing.

To try it before baking the URL in, leave `SHEET_API` empty and use **Sheet
settings** in the app — that stores the URL for your browser only.

### Export to a Google Sheet

**Export teams** now offers two destinations:

- **Excel file** — downloads an `.xlsx`. Works with no setup at all.
- **Google Sheet** — asks the script to create a *new* spreadsheet in Drive
  with the same eight tabs, and hands back the link. The master sheet is not
  touched; this is the Sheets equivalent of the download.

Exported copies land in a folder called **SGS Sports Randomizer exports**,
created next to the master sheet, and are named with the date, time and who
asked for them.

Two things to know. The script runs as whoever deployed it, so every export
lands in **that** person's Drive, not the clicker's. And by default the new
sheet is **private to the owner** — a committee member following the link will
see "request access" until it is shared. That default is deliberate: the export
carries residents' email addresses, phone numbers and flat numbers. To change
it, set `EXPORT_SHARING = 'anyone_view'` at the top of `Code.gs`.

Because this touches Drive, Google will ask for an extra permission the first
time. If you deployed the script before adding this, **re-deploy** (Deploy →
Manage deployments → edit → New version) and authorise again, or the Google
Sheet option will fail.

### Who did what

The link is public and there is no login, so the app asks for a name the first
time anyone adds, moves, swaps or deletes something, and remembers it in that
browser. Every change is stamped with it. Two extra tabs carry the record:

- **Manual entries** — everyone added through the app: name, gender, age group,
  flat, mobile, team, their events, and **who added them and when**. This is the
  tab to look at when you want just the people who were not in the form.
- **Change log** — one row per change: when, who, what kind (add / remove /
  move / swap / draw / import) and the detail, e.g.
  `Ananya Datt: Panthers → Titans`. It is **append-only** — the script adds
  rows it has not seen before and never rewrites or deletes existing ones, so
  the history survives even if someone opens the app in a fresh browser.

The team tabs also gain `Source`, `Added by` and `Added on` columns at the end.

Be clear-eyed about what this is: an honest record of who did what, not a
security control. Anyone can type any name. If you need it to be trustworthy,
that is the point at which you want real sign-in rather than a public link.

**What this does and does not give you**

- Every visitor sees the same teams, and the sheet is always current.
- Download it as a real `.xlsx` any time with **File → Download → Microsoft
  Excel** — or keep using the app's own **Export teams** button.
- Concurrent edits are merged, not clobbered (see above). The script also
  takes a lock so two writes cannot interleave mid-save.
- Anyone who has the link can save. If that becomes a problem, add a shared
  passphrase check at the top of `doPost` and a matching field in the app.
- The script only ever writes its own tabs (`Summary`, the four team tabs,
  `Unassigned`, and two hidden ones). Your form responses are untouched.

## What the app does

- **Balanced draw** — stratified by age group, then by gender inside each
  group, dealing each player to whichever team is currently lightest in that
  exact stratum. No age group is ever more than one player apart across teams.
- **Manual control** — swap two players, move one, delete one, or drag a row
  onto another team. Undo covers every change (Ctrl/Cmd+Z, 40 deep).
- **Add a player** — full registration fields plus a dynamic event picker;
  they go straight onto the team you choose.
- **Import** — any .xlsx/.csv registration export. Column detection is
  automatic and handles both the wide form-response shape and a long
  one-row-per-sport sheet.
- **Export** — a workbook with a summary tab, one tab per team carrying the
  registration sheet's own columns, a **Manual entries** tab and a **Change
  log** tab. Team rows end with `Source`, `Added by` and `Added on`.
- **Locked draws** — teams persist across reloads until you draw again or
  import a new roster.

## Adding a backend later

The app is deliberately self-contained, but two seams are ready for a server:

- **Roster** — `<script id="sheet-data" type="application/json">` near the
  bottom of `index.html` holds the registration sheet as `{headers, rows}`.
  Swap that block for a `fetch()` of the same JSON shape and nothing else
  changes.
- **State** — `save()` and `restore()` read and write one `localStorage` key
  (`sgs-sports-randomizer/v3`). The Google Sheet route above already layers a
  shared copy on top via `loadFromSheet()` / `saveToSheet()`; point those two
  at your own API instead if you outgrow Apps Script.

Two scripts load from CDN and must stay: SheetJS (import/export) and Google
Fonts. Everything else is inline.
