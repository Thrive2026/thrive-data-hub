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

## Who can do what

- **Viewing:** anyone with the link. There's no sign-in to view (an open item; see the ecosystem tracker).
- **Editing the roster:** click **Unlock Roster Editing** and sign in. Only Brandon, Avril and Sarah can unlock it. The allowlist is `EDIT_ALLOWLIST` in `index.html`. The sign-in lasts for that page visit only.
- **Crisis Center staff** are view-only here. Their records are edited in the Crisis Center hub (decision 9/23/26).

## Gotchas

- Redeploying with new people added to `SDEF` only adds new IDs to the live roster. It never overwrites someone who's already there. To correct an existing person, edit them in the app, or use a one-off migration script for bulk fixes.
- Name matching for imports goes through `normAliasName()` and the Supabase `staff_aliases` table. Bracketed notes such as "(HOLD)" are stripped automatically. A true nickname mismatch needs a `staff_aliases` entry.

## Links that point here

- The `thrive-hub` landing page (Data & Staff Hub tile)
- The Director Hub

