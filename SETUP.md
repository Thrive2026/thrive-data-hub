# Thrive Data & Staff Hub

*Updated 10/29/26.*

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

- **Viewing:** anyone with the link. There's no sign-in to view (an open item; see the ecosystem tracker).
- **Editing the roster:** click **Unlock Roster Editing** and sign in. Only Brandon, Avril and Sarah can unlock it. The allowlist is `EDIT_ALLOWLIST` in `index.html`. The sign-in lasts for that page visit only.
- **Crisis Center staff** are view-only here. Their records are edited in the Crisis Center hub (decision 9/23/26).

## Gotchas

- Redeploying with new people added to `SDEF` only adds new IDs to the live roster. It never overwrites someone who's already there. To correct an existing person, edit them in the app, or use a one-off migration script for bulk fixes.
- Name matching for imports goes through `normAliasName()` and the Supabase `staff_aliases` table. Bracketed notes such as "(HOLD)" are stripped automatically. A true nickname mismatch needs a `staff_aliases` entry.
- If the Linkage Hub shows "—" for Transitioned after an upload, the file was probably missing the clinical notes. Check that it was the org-wide export.

## Links that point here

- The `thrive-hub` landing page (Data & Staff Hub tile)
- The Director Hub
- The BH Linkage Hub (Optum referral upload link)

