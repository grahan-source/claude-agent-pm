# Signforge Admin — Architecture Reference

> **Purpose:** Fast-start context for every agent session. Read this before touching any code.
> **Repo:** `github.com/grahan-source/signforge-admin`

---

## Stack Overview

| Layer | Technology |
|---|---|
| Frontend | React + Vite + Tailwind CSS |
| Auth | Supabase Auth (magic link / passwordless email OTP only) |
| Database | Supabase Postgres |
| Storage | Supabase Storage |
| AI Tagging | Supabase Edge Function → Claude Sonnet (`claude-sonnet-4-5-20250929`) |

---

## Supabase Configuration

- **Project ref:** `infruhjobpvkkhekdkgw`
- **Region:** (check dashboard if needed)
- **Auth method:** Magic link only — no passwords, no OAuth
- **Storage bucket:** `sign-images`
  - Public bucket
  - 10MB file size limit
  - Accepted MIME types: `image/jpeg`, `image/png`, `image/webp`, `image/heic`

---

## Database Schema

### `signs` table

| Column | Type | Nullable | Notes |
|---|---|---|---|
| `id` | uuid | NO | `gen_random_uuid()` |
| `image_path` | text | NO | Path in Supabase Storage |
| `image_url` | text | YES | Public URL |
| `title` | text | YES | — |
| `source_type` | text | YES | — |
| `source_url` | text | YES | — |
| `product_category` | text | YES | — |
| `product_type` | text | YES | — |
| `aesthetics` | text[] | YES | Array |
| `color_approach` | text[] | YES | Array |
| `typography_style` | text[] | YES | Array |
| `complexity` | text | YES | — |
| `finish` | text[] | YES | Array |
| `dominant_colors` | text[] | YES | Array |
| `lighting` | text | YES | — |
| `installation` | text | YES | — |
| `size_category` | text | YES | — |
| `industries` | text[] | YES | Array |
| `ai_description` | text | YES | Free-text description from Claude |
| `ai_tags_raw` | jsonb | YES | Raw Claude response |
| `owner_notes` | text | YES | Manual notes |
| `quality_rating` | integer | YES | — |
| `is_featured` | boolean | YES | Default `false` |
| `is_approved` | boolean | YES | Default `false` |
| `embedding` | vector | YES | For future semantic search (pgvector) |
| `created_at` | timestamptz | YES | `now()` |
| `updated_at` | timestamptz | YES | `now()` |
| `created_by` | uuid | YES | References `auth.users(id)` ON DELETE SET NULL |

**Indexes:**
- `idx_signs_created_by` on `created_by`

---

## Row-Level Security

RLS is **enabled** on the `signs` table. Four granular policies, all scoped to `authenticated` role. Anon users get **nothing**.

| Policy name | Operation | Rule |
|---|---|---|
| `signs_select_authenticated` | SELECT | `USING (true)` |
| `signs_insert_authenticated` | INSERT | `WITH CHECK (true)` |
| `signs_update_authenticated` | UPDATE | `USING (true) WITH CHECK (true)` |
| `signs_delete_authenticated` | DELETE | `USING (true)` |

> **Note:** A previous catch-all policy named `authenticated_full_access` (ALL) was replaced with these four granular policies. If it reappears in a migration diff, it should be dropped.

---

## Edge Functions

### `tag-image`

- **Runtime:** Deno
- **Location:** `supabase/functions/tag-image/index.ts`
- **Model:** `claude-sonnet-4-5-20250929` (upgraded from Haiku)
- **What it does:**
  1. Receives an image path from the frontend
  2. Fetches the image from Supabase Storage
  3. Base64-encodes it
  4. Calls Claude Vision with a structured prompt
  5. Returns structured JSON tags matching the `signs` schema fields

---

## Frontend Structure

| File | Role |
|---|---|
| `App.jsx` | Auth wrapper, magic link login, routing |
| `UploadPage.jsx` | Drag & drop → triggers `tag-image` edge function → review form |
| `LibraryPage.jsx` | Browse/filter approved signs (category, aesthetic, industry, search), 3-column grid |
| `TagReviewForm.jsx` | Tag review/edit UI after AI tagging |
| `UploadZone.jsx` | Drag & drop upload component |
| `SignCard.jsx` | Card component — accepts `onUpdate` callback (edit UI not yet wired up) |

---

## Migration Convention

**All schema and RLS changes must be committed as migration files** in:

```
supabase/migrations/YYYYMMDDHHMMSS_description.sql
```

Do not apply schema changes via the Supabase dashboard without committing the corresponding migration file. The repo is the source of truth. If a change exists in Supabase but not in `supabase/migrations/`, the first task should be to write and commit the missing migration.

---

## Repo State (as of 2026-03-04)

- 2 commits: `Initial commit` → `Fix edge function`
- RLS policies above were applied server-side but **not yet committed** as a migration file — this is Task #1 in `TODOS.md`
- No other local-only changes

---

## Future Work (not yet started)

- **Vector search / semantic similarity:** `embedding` column exists in schema, pgvector needs to be enabled, embedding generation needs to be added to the `tag-image` edge function
- **Style matching:** Match signs against customer text descriptions (depends on embeddings)
