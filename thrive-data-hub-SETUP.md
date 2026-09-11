# Thrive Data & Staff Hub

Single-file app (`index.html`) — the shared staff roster and central data tooling for the Thrive Tools ecosystem. Deployed as a static site on Render, auto-deploying from this repo's `main` branch.

## Deploy

1. New Render Static Site, connect this GitHub repo (`Thrive2026/thrive-data-hub`).
2. Build command: none. Publish directory: `/` (repo root — `index.html` is served directly).
3. No environment variables needed — the Supabase URL and anon key are embedded in the page, same as every other Thrive Tools hub.

## What this is

This was split out of the `thrive-hub` repo (previously `data_staff_hub.html`) on 9/11/26, because it's ecosystem-wide infrastructure — the shared staff roster and data-ingestion tooling every hub reads from — not a Direct Service Program tool. Same Supabase project as the rest of the ecosystem (`fussjixekyhwuromauff.supabase.co`), reading/writing the `appdata` key/value table, primarily the `staff` key that every other hub reads.

## Cross-repo links that point here

Once this is live, update these hubs' links from the old relative path to this deployment's real URL:
- `thrive-hub`'s landing page (`index.html`) — the Data & Staff Hub tile.

If any other hub is later found to link here, update it the same way.
