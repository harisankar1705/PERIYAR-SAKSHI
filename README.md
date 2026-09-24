# PERIYAR SAKSHI — Frontend (Citizen Portal + Admin Console)

Two single-file HTML apps that share one Supabase project:

| File | Role |
|---|---|
| `PERIYAR_SAKSHI_LAST_PRO.html` | Citizen-facing portal — report a sighting, view impact, mini admin tab |
| `ADMIN_REWORKED.html` | Dedicated authority console — triage, verify, dispatch, evidence review |

Both already point at the same project:
```js
const SUPABASE_URL = 'https://dvuestkbydcpikivwqti.supabase.co';
const SUPABASE_KEY = 'sb_publishable_...';
```
That's what "connects" them — every report the citizen portal writes, the
admin console reads, and vice versa for status updates. **This was already
true before this change.** What wasn't wired up until now was the evidence
photo: it was previewed locally in the browser but never actually sent
anywhere (`p_photo_path: null` was hardcoded).

## What changed in this pass

1. **Real photo upload.** The citizen portal now uploads the attached photo
   to a Supabase Storage bucket (`evidence-photos`) when a report is
   submitted, and passes its public URL to the `create_report_with_impact`
   RPC instead of `null`.
2. **Photo shows up on both sides.** Both files now read `photo_path` back
   from the `reports` table and render a thumbnail wherever a report is
   shown (citizen impact panel, citizen's own mini admin tab, and the
   dedicated admin console's triage cards).
3. **Download Evidence button.** Both files now have a `window.downloadEvidence(reportId)`
   function — it fetches the image as a blob and forces a real browser
   download (not just "open in new tab"), which is what actually makes a
   cross-origin Supabase Storage image save to disk reliably.

See `docs/SUPABASE_SETUP.md` — **you need to do two things in your Supabase
project before uploads will work**: create the storage bucket, and add the
`photo_path` parameter to the `create_report_with_impact` function. Nothing
in the HTML can do that for you; it's server-side config.

## Local testing without touching Supabase

Both files already degrade gracefully when `supabase` is `null` (e.g. the
CDN script didn't load, or you're testing offline) — reports just stay in
the local `reportsList`/`adminReports` array for that browser tab and the
photo preview still works (it just won't be uploaded or downloadable by the
other file, since there's nothing shared to sync from).

## Keeping docs in sync

If you change the Supabase schema, RPC signature, or bucket name again,
update `docs/SUPABASE_SETUP.md` in the same change — it's the only place
that documents what the backend is expected to look like for these two
files to work together.
