# Boom Sports — TV Web App

A single-page TV webapp: **Login screen → full-screen live channel/league player**.
Built to be watched, not clicked around in — the whole screen *is* the player, with
channel/league selection living inside an overlay that auto-hides after 5 seconds
and reappears on any remote press.

## What's inside
- `index.html` — the entire app (HTML + CSS + JS in one file, no build step)
- TV remote navigation: Arrow keys move focus between tiles, Enter/OK selects,
  Back (Escape/Backspace) dismisses the overlay early
- Supabase email/password auth (sign in **and** sign up, so you can create a
  test account immediately)
- Reads your `sports_hub` table's `data` jsonb column and builds channel/league
  tiles from it, following the same key pattern as your Flutter app
  (`{Key}` = logo, `{Key}_Video}` = stream, `{Key}_ClickCount`, `{Key}_DisplayName`, etc.)
- HLS playback via hls.js for `.m3u8` streams; falls back to a direct `<video>` src otherwise
- No live-match/fixture data is used — MatchId fields are intentionally ignored

## Deploy to Render (free static site)

1. Put `index.html` in its own GitHub repo (or a folder in an existing one).
2. Go to [render.com](https://render.com) → **New +** → **Static Site**.
3. Connect the repo.
4. Build command: *(leave blank)*
5. Publish directory: `.` (or wherever `index.html` lives)
6. Click **Create Static Site** — Render gives you a live
   `https://your-app-name.onrender.com` URL within a minute or two.
7. Open that URL on the Smart TV's browser to test remote navigation for real.

No environment variables are needed — the Supabase URL and anon key are already
in the file (this is normal and safe for the anon/public key).

## Swapping in your real logo

Right now the header uses a text wordmark (`BOOM` in white + `SPORTS` in red).
To use your actual logo image instead:
1. Add your logo file to the repo, e.g. `logo.png`.
2. In `index.html`, replace both:
   ```html
   <div class="brand-logo display"><span class="boom">BOOM</span><span class="sports">SPORTS</span></div>
   ```
   with:
   ```html
   <img src="logo.png" alt="Boom Sports" style="height:64px;">
   ```
   (there are two spots — one in the login card, one in the top-left of the home overlay;
   use a smaller height, e.g. `34px`, for the home-screen one).

## One thing to check in Supabase

The "view count" ticking up when someone selects a channel is a best-effort write
back to your `sports_hub` table. If your Row Level Security policies only allow
`admin` writes (as your schema notes suggest), that write will silently fail —
which is fine, playback isn't affected either way. If you want anonymous viewers
to be able to bump the click count, you'd need an RLS policy permitting `UPDATE`
on that table for the `anon` role, scoped narrowly (e.g. via a Postgres function
that only increments counters, not a raw table-wide update).
