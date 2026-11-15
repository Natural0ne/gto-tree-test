# GTO Talent · Soul · Trait Builder

A single-page app for building and tracking characters in the Grand Terra Online TTRPG homebrew.

## File structure

```
index.html              The app (HTML, CSS, JS).
config.json             Cloud sync credentials (optional, see below).
config.example.json     Template to copy.
schema.sql              One-time Supabase setup SQL.
data/                   All game content, as JSON. Edit to add talents,
                        effects, synergies, etc. — no code changes needed.
  flags.json            Story flags shared by talents and effects.
  talents.json          The Talent Tree.
  soul-categories.json  Mechanical-kind tags for Soul Effects.
  soul-themes.json      Thematic tags that drive synergies.
  soul-effects.json     The Soul Effects library.
  soul-synergies.json   Synergy groups.
  soul-slot-growth.json Auto slot gains at certain levels + flag-gated slots.
  roles.json            Role chassis (hit die, saves, universal features).
  backgrounds.json      Heritages / backgrounds, including legendary ones.
  classes.json          Class list (metadata).
  named-souls.json      Named Souls (e.g. Morning Star) with staged effects.
```

## Hosting

Any static host works. Upload the whole folder.

### GitHub Pages (recommended)

1. Create a public repo (any name — `gto-tree` works).
2. Upload all files to the root of the repo (`index.html`, `data/`, `config.json` once set up).
3. In the repo's **Settings → Pages**, enable Pages from the main branch.
4. Visit `https://<username>.github.io/<repo>/` in a minute or so.

### Updating content

1. Edit a JSON file in your repo (via GitHub's web editor or locally).
2. Commit and push.
3. Players refresh the page to see the new content.

The app fetches JSON with `cache: 'no-cache'`, so updates appear on the next page load.

## Local testing

```
cd gto-site
python3 -m http.server 8000
```

Then open `http://localhost:8000/` in your browser. You cannot just double-click
`index.html` — browsers block `fetch()` on `file://` URLs.

## Player builds

Each build saves to the browser's localStorage by default. The Export and
Import buttons work at any time: Export writes a JSON file, Import reloads one.
This is the safety net; the cloud sync below is the sharing layer.

---

# Cloud sync setup (Phase 1)

The app can save builds to a Supabase database so you and your players share
one source of truth across devices. Setup takes about 10 minutes, once.

## 1. Make a Supabase project

1. Go to https://supabase.com and sign in (free tier is enough).
2. **New project**. Pick a name, pick a region near your group, set a database
   password (save it — you won't need it for the app but Supabase requires one).
3. Wait a minute for the project to provision.

## 2. Create the tables

1. In the Supabase dashboard, open **SQL Editor** → **New query**.
2. Paste the contents of `schema.sql` into the editor.
3. Click **Run**. You should see "Success. No rows returned."

This creates two tables (`campaigns`, `builds`) and opens them to anonymous
reads and writes via the project's anon key. For a small trusted group this
is appropriate; it is not secure against a stranger who gets the anon key.

## 3. Get your credentials

1. In the dashboard, open **Project Settings** (gear icon) → **API**.
2. Copy two values:
   - **Project URL** (looks like `https://abcdefghijklmnop.supabase.co`)
   - **anon public key** (a long string starting `eyJ...`)

## 4. Drop them into `config.json`

1. Copy `config.example.json` to `config.json`.
2. Edit it:

```json
{
  "supabase": {
    "url": "https://abcdefghijklmnop.supabase.co",
    "anonKey": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

3. Commit and push. (If the repo is public, the anon key will be public. That is
   the intended model for Phase 1: share codes are identity, not the anon key.
   Rotate the key in Supabase's dashboard if you want to invalidate access.)

## How it works

### Players

1. Visit the site.
2. Click the **Local / Sign in** button (top-right, teal when cloud is available).
3. Enter the campaign code the DM shares with the group.
4. Leave the build code blank for a new build, or paste a build code you were
   given to load a specific build.
5. Click **Connect**. Any change to your build auto-saves to the cloud within
   a second.

Share a build with someone (another device, the DM, a friend): hit **Copy
Share Link**. The link embeds the campaign and build codes.

### DMs

Visit the site with `?dm=<campaign_code>` in the URL:

```
https://yourname.github.io/gto-tree/?dm=DRAGON
```

A new **DM** tab appears. It shows every build in that campaign with level,
role, heritage, class(es), talent count, Soul, flags, and last-updated time.
Each build has:

- **Open in new tab** — opens the player's build in a fresh tab for a close look.
- **View as this build** — loads it into your current tabs. Any change you make
  will save back as that build (Phase 3 will add a read-only mode).

### Codes

Codes are short strings of uppercase letters and digits (no ambiguous
characters). They act as identity: anyone with a build code can read or
overwrite that build, and anyone with the campaign code can list every build
in it. This is deliberate for a small, trusted group.

---

## Troubleshooting

- **Cloud button says "Local"**: `config.json` is missing or doesn't have both
  `supabase.url` and `supabase.anonKey`. The app still works, just no sync.
- **Cloud button says "Sync error"**: open the Cloud panel to see the error.
  Common causes: wrong URL, wrong anon key, the tables weren't created, or
  Supabase paused the project (free tier does this after a week of no traffic;
  a single API call wakes it back up).
- **Players see different builds under the same code**: they're using different
  share codes. The build code is identity; it decides which row they edit.
