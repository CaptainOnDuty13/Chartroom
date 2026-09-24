# Wake Atlas 🧭

A private, self-hosted atlas for GPS tracks — import KML files, file them under trips and countries, and browse them on a real map. Built as an independent archive that doesn't depend on Google Earth (or any single tool) staying available or fast.

**Live site:** https://captainonduty13.github.io/Chartroom/

---

## Why this exists

Google Earth is heavy, its desktop client is being phased out, and a lifetime of GPS tracks shouldn't live somewhere that might not be there in a few years. Wake Atlas is a small, personal alternative: a single page you host yourself, backed by a database you control, that does one thing — store and browse your own tracks — without needing anything else installed.

## Features

- **Import KML** — drag and drop one or more `.kml` files. Bundled exports (a Google Earth "My Places" folder with many tracks in one file) are automatically split into individual tracks, each keeping its own name and original color. Raw per-sample GPS points — the "Track Points" folder Garmin exports include — are recognized and skipped, so a single activity imports as one track rather than hundreds.
- **Organize by trip or country** — tag each import, and switch the sidebar between grouping views at any time. A track can belong to more than one country.
- **Real map tiles** — powered by MapTiler, with a choice of Streets, Outdoor, or Satellite styles.
- **Per-track and per-group actions** — rename, show/hide, delete, and export (KML or GPX) either one track at a time or an entire trip/country as a single bundled file.
- **Day / night chart themes**, a collapsible sidebar, and a mobile-friendly layout.
- **Private by default** — the app sits behind a login; only an authenticated session can read or write data.

## How it's built

| Piece | What it does |
|---|---|
| **Frontend** | A single self-contained `index.html` — vanilla JavaScript, [Leaflet](https://leafletjs.com/) for the map, no build step, no framework. |
| **[Supabase](https://supabase.com/)** | Postgres database (`tracks` table), file storage (original `.kml` uploads), and authentication. Row-level security restricts every table and bucket to authenticated requests only. |
| **[MapTiler](https://www.maptiler.com/)** | Basemap tiles (Streets / Outdoor / Satellite), free tier. |
| **GitHub Pages** | Static hosting, deployed straight from this repo. |

Nothing is compiled or bundled — the whole app is one HTML file you can open, read, and edit directly.

## Setting this up for yourself

If you're standing up your own copy rather than using the one above:

1. **Create a Supabase project.** In the SQL editor, create the schema:
   ```sql
   create table tracks (
     id uuid primary key default gen_random_uuid(),
     label text not null,
     trip text,
     countries text[] default '{}',
     color text,
     kml_file_url text,
     geojson jsonb not null,
     created_at timestamptz default now()
   );

   alter table tracks enable row level security;

   create policy "authenticated full access" on tracks
     for all to authenticated using (true) with check (true);
   ```
   Then, under **Storage**, create a private bucket named `kml-files`, with read/insert/delete policies on `storage.objects` scoped to `bucket_id = 'kml-files'` and the `authenticated` role.
2. **Create your login.** Under **Authentication → Users**, add yourself as a user (email + password). This is the only account the app expects.
3. **Get a MapTiler key.** Free account at [cloud.maptiler.com](https://cloud.maptiler.com/account/keys/) — no card required.
4. **Wire up the frontend.** In `index.html`, set `SUPABASE_URL`, `SUPABASE_KEY` (the *publishable* key — safe to expose, since RLS is what actually protects the data), and `MAPTILER_KEY` near the top of the `<script>` block.
5. **Deploy.** Push to a public GitHub repo, then enable **Settings → Pages → Deploy from a branch → main → / (root)**. GitHub Pages requires a public repo on the free plan; that's fine here, since nothing secret lives in the code — the publishable key is meant to be public, and RLS is what actually guards the data.

## Using it day to day

- **Add a track:** the **+ Add track** button opens a file picker — choose one or more `.kml` files, optionally tag them with a trip name and one or more countries (comma-separated), and add. Bundled exports become several tracks automatically.
- **Browse:** the sidebar groups tracks by trip or country (toggle at the top); each group starts collapsed — click to expand. The checkbox next to a group's name shows or hides every track in it at once.
- **Export:** the ⬇ button — on a single track, or on a whole group — offers KML or GPX. A group export bundles every track in it into one file, so a whole trip or country can be pulled out as a single archive.
- **Rename / delete:** the ✎ and 🗑/✕ icons on both individual tracks and whole groups.
- **Map style:** the **Map style** button in the header switches between Streets, Outdoor, and Satellite tiles.

## Known limitations

- Altitude isn't captured from imported KML files (only latitude/longitude), so it won't appear in exports either, even if the original file had it.
- Native KML colors are read per-placemark on import; if a file has no style information at all, tracks fall back to an auto-assigned color palette.
- This is a single-user app by design — one login, no per-user data separation.

## Version

**1.1.3** — see the version comment at the top of `index.html`.

- 1.1.3 — three import/display fixes: raw GPS sample points in a "Track Points" folder (as Garmin exports them) are no longer imported as hundreds of separate tracks; coordinates written with a space after the comma (`103.79, 1.28`) now parse correctly instead of landing far from their true position; and the sidebar collapse no longer blanks the map.
- 1.1.2 — first attempt at the sidebar-collapse map fix (told Leaflet to re-measure its container; superseded by the 1.1.3 CSS fix).
- 1.1.1 — corrected the Satellite map style from the deprecated `satellite-v2` tile ID to the current `satellite-v4`.
- 1.1.0 — collapsible sidebar, batch group actions (select-all, export, delete), default-collapsed trip/country groups, MapTiler map-style picker (Streets/Outdoor/Satellite), header reorganization, mobile layering fix, versioning and this README.

---

Built collaboratively with Claude (Anthropic), in conversation, over several sessions.
