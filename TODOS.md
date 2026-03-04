# Signforge Admin — Project Backlog

> Managed via claude-agent-pm workflow.
> PM Agent: scopes work, writes specs in `specs/`, dispatches Coding Agent.
> Coding Agent: works in `github.com/grahan-source/signforge-admin`, commits and pushes.
> QA Agent: tests, PM Agent verifies and marks done.
> Architecture reference: `docs/ARCHITECTURE.md`

---

## Status Legend

- `[ ]` Pending
- `[~]` In Progress
- `[x]` Done
- `[!]` Blocked

---

## Bundle 0 — Repo Sync (prerequisite for everything)

> Get the GitHub repo into a known good state so the agent always works from ground truth.

- [ ] **T0.1 — Commit RLS migration**
  Write and commit `supabase/migrations/20260304000000_rls_policies.sql` capturing the four granular RLS policies currently applied server-side but missing from the repo. Drop the old `authenticated_full_access` catch-all if present. See `docs/ARCHITECTURE.md` for exact policy definitions. Push to `main`.

- [ ] **T0.2 — Upgrade edge function model to Sonnet**
  In `supabase/functions/tag-image/index.ts`, replace `claude-haiku-4-5-20251001` with `claude-sonnet-4-5-20250929`. Commit and push. Verify the function still deploys cleanly.

---

## Bundle 1 — Edit Existing Library Entries (P1)

> `SignCard.jsx` has an `onUpdate` callback wired up but no edit UI. Editing tags on approved signs is needed for data quality.

- [ ] **T1.1 — Spec: Edit tags on library entries**
  PM Agent writes spec covering: inline edit mode on SignCard, which fields are editable (all tag fields + owner_notes + quality_rating + is_featured + is_approved), save/cancel flow, optimistic UI update, Supabase `update()` call with `updated_at` timestamp.

- [ ] **T1.2 — Implement edit UI**
  Coding Agent implements per spec. Fields: all tag columns from schema. Requires updating `SignCard.jsx`, `LibraryPage.jsx`, and potentially extracting an `EditModal.jsx` or `EditDrawer.jsx` component.

- [ ] **T1.3 — QA edit flow**
  Test: edit a tag, save, verify update reflected in library. Test: cancel edit, verify no change. Test: edit `is_approved` and `is_featured` toggles.

---

## Bundle 2 — Bulk Upload Queue (P1)

> Single-file upload works. README flags bulk upload as the next major feature.

- [ ] **T2.1 — Spec: Bulk upload queue**
  PM Agent writes spec covering: multi-file selection in `UploadZone.jsx`, queue state management (pending / processing / done / error per file), sequential calls to `tag-image` edge function, progress display, error handling per file without aborting the queue.

- [ ] **T2.2 — Implement bulk upload**
  Coding Agent implements per spec. Likely requires refactoring `UploadPage.jsx` state management significantly.

- [ ] **T2.3 — QA bulk upload**
  Test: upload 5 images, verify all tagged sequentially. Test: one bad file in queue, verify others complete. Test: progress UI updates correctly.

---

## Bundle 3 — Library UX Improvements (P2)

- [ ] **T3.1 — Pagination**
  `LibraryPage.jsx` fetches with a `limit` but no pagination UI. Add "Load more" or page controls. Keep current filter state across pages.

- [ ] **T3.2 — Featured flag toggle**
  `is_featured` exists in schema but no UI to toggle it. Add toggle to `SignCard.jsx` (visible in library, behind `is_approved` gate or always visible — PM to decide in spec).

---

## Bundle 4 — Vector Search / Semantic Similarity (P3)

> Higher effort. Needs pgvector enabled in Supabase + embedding generation in the edge function.

- [ ] **T4.1 — Spec: Embedding pipeline**
  PM Agent scopes enabling pgvector, adding embedding generation to `tag-image` (use `ai_description` text as input), storing in `embedding` column, building a similarity search endpoint.

- [ ] **T4.2 — Implement embedding pipeline**

- [ ] **T4.3 — Style matching UI**
  Allow users to input a text description and find similar signs by vector similarity. This is the primary end-goal use case.

---

## Completed

_(none yet)_
