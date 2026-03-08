# QA Handoff — Bundle 0: Repo Sync

## What Changed
- Committed RLS migration (`20260304000000_rls_policies.sql`) capturing four granular policies for the `signs` table
- Upgraded `tag-image` edge function model from `claude-haiku-4-5-20251001` to `claude-sonnet-4-5-20250929`

## Commits
- `7bb6917` — Add RLS policies for signs table
- `4916593` — Upgrade tag-image edge function model from Haiku to Sonnet

## Verified
- [x] Migration file exists at `supabase/migrations/20260304000000_rls_policies.sql`
- [x] Edge function uses `claude-sonnet-4-5-20250929` (confirmed in `supabase/functions/tag-image/index.ts:89`)
- [x] Local and remote (`origin/master`) are in sync — clean working tree

## Notes
- Repo default branch is `master`, not `main`
- No regressions expected — these were sync/housekeeping tasks only
