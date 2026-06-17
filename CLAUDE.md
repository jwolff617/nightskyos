# Night_OS (standalone repo)

This is the standalone Night_OS deployment (jwolff617/nightskyos.git).
It deploys independently AND is also embedded inside SageDS as an iframe.

## SYNC RULE — ALWAYS UPDATE BOTH FILES

When the user asks to update Night_OS, **always update both**:
1. `C:\Users\sagep\Downloads\NightOS\index.html` (this repo — standalone deployment)
2. `C:\Users\sagep\Downloads\SageOS\nightos\index.html` (SageDS-embedded copy)

Apply the change to both files in the same session. Commit and push each repo separately
(`NightOS` pushes to jwolff617/nightskyos.git, `SageDS` pushes to jwolff617/sageos.git).

Exception: changes that are SageDS-specific (EMBED_MODE behavior, postMessage bridge,
exchange-token auth) only belong in `SageOS/nightos/index.html`, not here.
Exception: changes that are standalone-only (non-embed header, theme toggle visible)
only belong in this file.

When in doubt, apply to both and note which parts are embed-conditional behind `EMBED_MODE`.

## How Night_OS embeds inside SageDS

The SageDS shell loads this app at `/nightos/?embed=1`. In embed mode:
- `EMBED_MODE = true` hides the shell chrome (header, map, command bar, theme toggle)
- Auth comes via `postMessage` from SageDS parent (not Supabase magic link)
- The token is an HS256 JWT signed by SageDS's `exchange-token` Edge Function

See `SageDS/nightos/CLAUDE.md` for the full postMessage protocol and SageDS security rules.

## Security rules (non-negotiable)

- The Night_OS **service role key** must NEVER appear in client-side code.
  It lives only in Supabase Edge Function env vars (set via Supabase dashboard → Edge Functions → Secrets).
- Only the anon key (`eyJ...`) is safe in this file.
- RLS is ON for every table. Never disable it.
- Night_OS FK constraints on auth.users were dropped (nodes, journal_entries, archive_rows,
  user_settings, eod_log, collab_invites) — SageDS UUIDs don't exist in Night_OS auth.users.
  RLS via `auth.uid()` is the real security layer.

## CDN pinning rule

Every `<script src>` must pin an exact version — no bare `@major` or unversioned URLs.
Current pins:
- `@babel/standalone@7.29.7` (v8.0.0 was a breaking release; never unpin this)

## Night_OS Supabase project

- URL: `https://gcsfupdmyubwpoxbvnru.supabase.co`
- Separate from SageDS Supabase (`zqaupsxwcplvhozostys`) — data stays here, never centralized

## Known issues (as of 2026-06-16)

- `user_settings` upsert returns 400 — `sync_seed_to_context` column missing from live table.
  Fix: `ALTER TABLE user_settings ADD COLUMN IF NOT EXISTS sync_seed_to_context boolean DEFAULT false;`
  Run in Night_OS Supabase SQL editor. Also verify unique constraint on `user_id` exists.
