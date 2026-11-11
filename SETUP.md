# Thrive Data & Staff Hub

*Updated 11/9/26.*

A single-file app (`index.html`) that holds the shared staff roster and the central data uploads for the Thrive Tools ecosystem. It runs as a static site on Render and redeploys automatically from this repo's `main` branch.

**Live URL:** `https://thrive-data-hub.onrender.com`

## Deploy

1. Create a new Render Static Site and connect this repo (`Thrive2026/thrive-data-hub`).
2. Leave the build command empty. Set the publish directory to the repo root (`index.html` is served directly).
3. No environment variables are needed. The Supabase URL and anon key are embedded in the page, as in every other Thrive Tools hub.

## What this is

This was split out of the `thrive-hub` repo on 9/11/26 (it used to be `data_staff_hub.html`). It's ecosystem-wide infrastructure, not a program hub. It uses the shared Supabase project (`fussjixekyhwuromauff`):

- **The `staff` roster** is in the `appdata` key `staff`, which every other hub reads. This is the one place it's edited, except Crisis Center staff (see below).
- **Central uploads** are the recurring Pimsy, Klip and QuickBooks exports (Pimsy Notes, renewals, supervision, productivity, linkage, the Program Hub ledger and so on). They're written to `appdata` keys that the other hubs read live.
- **SLI Screeners (ongoing):** drop in the Pimsy Screeners export. SLI 2.0 is scored in Pimsy, and this upload copies those results into the `sli_assessments` table, which feeds the Flagged Notes Tracker. Requires sign-in.

## Pimsy Notes → BH Linkage Hub

The Pimsy Notes upload is the only data source for the Linkage Hub. It must be the **org-wide** Notes export (the same file Clinical Goals reads), not a Linkage-only pull. The Linkage conversion count needs clinical staff's notes in the file.

On upload it writes three `appdata` keys:

- `linkage_notes_raw`: every note row, stored per month and keyed by Note #. Re-uploading notes that are already stored doesn't double-count them, so weekly delta files and overlapping exports are safe.
- `linkage_history`: one computed snapshot per month (Buckeye U-codes, sites, payers, providers, non-Buckeye counts).
- `linkage_data`: the latest month, plus `nonBkSinceCutover` (below).

### All Other Medicaid conversion (added 10/29/26)

`computeNonBkConversions()` runs over the whole raw pool after each upload, because a conversion often spans months. For example, a client touched in July might be assessed in August.

- **Touched:** a non-Buckeye client with a Linkage note on or after 7/1/26. Each person is counted once across the whole period. This replaced the old method of adding up monthly unique counts, which counted repeat clients more than once.
- **Converted ("Transitioned"):** that client has a note billed H0001, 90791, or another clinical code on or after the day of their first Linkage contact. "Clinical code" uses the same test as Clinical Goals: any code in `CLINICAL_RATE_TABLE` (90791/90832/90834/90837), or CPST, plus H0001. A code added to `CLINICAL_RATE_TABLE` is counted here automatically.
- **Already in clinical care:** a client who had a clinical note before their first Linkage contact. These are excluded from the rate and reported separately. This only reaches back as far as the history in `linkage_notes_raw`.
- **Rate:** converted ÷ (touched − already in clinical care).

Each post-cutover month in `linkage_history` also gets `nonBk.converted` and `nonBk.touched_new`, so monthly views add up correctly.

**After changing this logic, re-upload any recent Notes export.** The numbers are only recomputed on upload.

## Who can do what

- **Viewing:** the whole page requires sign-in (added 10/29/26). Only org-wide reviewers (`reviewers.role = org_wide`) can get in (Brandon, 10/29/26; change `DS_VIEW_ROLES` in `index.html` to widen it). Nothing loads from Supabase until then. The sign-in is held in memory, so it lasts for that page visit only, and reloading asks again.
- **Editing the roster:** after signing in, click **Unlock Roster Editing**. It checks the signed-in account against `EDIT_ALLOWLIST` in `index.html` (Brandon, Avril, Sarah Neff). No second password is needed.
- **Data requests** use the signed-in person's own login rather than the public key, so the hub is ready for the appdata lockdown.
- **Crisis Center staff** are view-only here. Their records are edited in the Crisis Center hub (decision 9/23/26).

## Community hours and engagement from raw data (added 11/9/26)

Billable hours, caseload, outreach % and engagement % are now computed in this hub from uploads you already make. The **Productivity workbook** and the **Caseload Engagement report** are no longer needed. Both upload boxes have been removed, here and in Community Hub. Enter Data now shows an "Automatic — nothing to upload" card with how current the hours and engagement data are, plus **↻ Recompute now**.

- **Billable hours** come from the Pimsy Notes upload. Every note counts except Non-billable, ADMIN, OTR- Linkage, CTP ADMIN/Engagement/In-Reach, ITP and Phone. Periods run Saturday to Friday, two weeks at a time (e.g. 10/24–11/06), and are stored under the period's end date, as the old tabs were. The Notes pool now keeps each note's Duration in Minutes (`m`) for this.
- **Caseload** comes from the Renewals upload: each client counts toward their Peer Supporter. Each Renewals upload saves a snapshot to the `appdata` key `caseload_snapshots` (client numbers one-way hashed, last 8 kept). A client whose first note is after the week isn't counted in that week's caseload.
- **Outreach** means the client had any note that week, from anyone. **Engagement** means the client had an H0038 or H0036 note that week. Weeks run Monday to Sunday and are stored under the Sunday.
- **Late notes count** (Brandon, 11/9/26). A note released late doesn't count against anyone's hours or engagement. Lateness shows up only in the Late Notes metric. Each upload recomputes the last 6 two-week periods and the last 8 weeks, so late notes are picked up when they arrive.
- **When it runs:** automatically after a Pimsy Notes import and after a Renewals import, or with **↻ Recompute now** on Enter Data. A toast lists what was updated and anything skipped, with the reason.
- **Upload Renewals early in the week.** Caseload for a week uses the first Renewals snapshot taken on or after that week ends, within 14 days.
- **First run after deploying:** notes uploaded before 11/9/26 don't have minutes, so a period is skipped until a Notes export covering it is re-uploaded. Re-upload one export covering 10/24 onward.
- **Checked against the old reports (11/9/26):** hours matched the 10/10–10/23 report exactly for 66 of 73 staff. The differences were notes released after that report was run. Engagement flags matched for 93–99% of clients, with misses from late-released notes.
- **UKG** (PTO and overtime) isn't part of this. The hubs don't use PTO today.

## UKG time: PTO, overtime and Productivity % (added 11/11/26)

Upload both UKG "Calculated Time Summary" reports for the same pay period together, using the **⏱ UKG Time** box on Enter Data: *Community Productivity Report* and *Total Hours for Vlookup - Community Productivity Report*. Emily sends both every other Monday.

- **Pay period:** read from the report's Counter Date and stored under the period's end date, the same key billable hours use.
- **How the two files combine:** the Vlookup report covers every non-exempt employee across all cost centers, including ADMIN, so it's used for anyone in it. The adult-only report fills in anyone it leaves out (full-time exempt staff). Staff are matched by Employee Id.
- **Hours for Calculation** uses the old productivity workbook's formula:
  - Full-time: 80 + overtime − admin/meeting hours (ADMIN cost center) − PTO − floating holiday.
  - Part-time/split: regular hours on adult cost centers + overtime + holiday worked 1.0 and 1.5.
- **Stored in `pd`** as `hfc`, `pto`, `ot` and `adminHrs`.
- **Community Hub** gets a **Productivity** column (billable hours ÷ Hours for Calculation, with a PTO badge and a hover breakdown). When UKG is loaded for the same pay period as the hours, both the hours target and Progress to Goal are scaled by Hours for Calculation ÷ 80 (FT) or 40 (PT), so PTO lowers what's expected instead of counting against the person. Periods without UKG show "—" and use the flat targets.
- **Checked 11/11/26** on the 10/24–11/06 reports: Willie Richardson 60.88 billable ÷ 76.28 = 80% (8 hours of PTO); Earl McDonald 83%; Kimberly Paridon 65%.

### What each hub has (added 11/9/26)

The checklist has two tabs. **What to upload** lists each file. **What each hub has** shows one card per hub (Community, Hospital, Courts & Jail, Linkage, Director, Clinical Director, Executive, Outcomes, Flagged Notes Tracker, BD Dashboard, Data & Staff Hub). Each card lists every dataset that hub shows, how far its data goes, and which upload it comes from, with a green, amber or red dot. The hub's badge (Up to date / Due soon / Old or missing data) is its worst dataset, and hubs with old data are listed first. A stale dataset has an **Upload ↓** button that jumps to the right upload box. To change which hub shows what, edit `HUB_FEEDS`, or `HUB_DATASETS` for the dataset names, in `index.html`.

## Late Notes from raw data (added 11/9/26)

The **Late Notes 2026 workbook** isn't needed anymore. Its upload box has been removed here and in Community Hub. Late notes are calculated from the Pimsy Notes upload using the same rule as the workbook:

- Peer support notes only (H0038, H0038 GT, H0038 HM).
- "Hours late starting the note" = **Created Date minus Start Note By Date**. Over 48 hours counts as late.
- Grouped by session date into Saturday–Friday weeks, stored under the Friday (as the old "10.24-10.30" tabs were).
- Each note keeps its hours late in the notes pool as `l`. A note that hasn't been started yet can't be judged; it's counted on a later upload, because the last 8 weeks are recalculated every time.
- Notes uploaded before 11/9/26 don't have this field, so earlier weeks are skipped until a Notes export covering them is re-uploaded.
- **Check after deploying:** compare the calculated week of 10/24–10/30 with the "10.24-10.30" tab of the old workbook. On 11/9/26 the formula matched the workbook's own columns, but no Notes export covering that week was on hand for a person-by-person check.

## Upload checklist (added 11/9/26)

The top of **Enter Data** is now a checklist of every file that still has to be uploaded by hand. Each row shows how often it's due, when it was last uploaded and by whom, how far the data goes, a Fresh / Due / Overdue / Never uploaded badge, where to get the file, and which hubs it feeds. Overdue files are listed first. **Upload ↓** jumps to that file's upload box. Below the list is a section of what's now automatic and no longer needs an upload. Status comes from the same import registry as the Data Catalog tab, so the two always agree. To add or change a row, edit `UPLOAD_CHECKLIST` in `index.html`. Renewals is now due weekly instead of every two weeks, because caseload for engagement % depends on it. TripLog and JotForm mileage aren't on the checklist yet because their uploads don't record when they were last done.

## Gotchas

- Redeploying with new people added to `SDEF` only adds new IDs to the live roster. It never overwrites someone who's already there. To correct an existing person, edit them in the app, or use a one-off migration script for bulk fixes.
- Name matching for imports goes through `normAliasName()` and the Supabase `staff_aliases` table. Bracketed notes such as "(HOLD)" are stripped automatically. A true nickname mismatch needs a `staff_aliases` entry.
- If the Linkage Hub shows "—" for Transitioned after an upload, the file was probably missing the clinical notes. Check that it was the org-wide export.
- `linkage_notes_raw` holds every org-wide note row (client #, insurance, provider) and grows each month. It's kept out of the browser's local storage on purpose. Other pages shouldn't load all of `appdata` at once (Community Hub and Provider Match now skip or exclude it). Any new page should request only the keys it needs.

## Links that point here

- The `thrive-hub` landing page (Data & Staff Hub tile)
- The Director Hub
- The BH Linkage Hub (Optum referral upload link)



