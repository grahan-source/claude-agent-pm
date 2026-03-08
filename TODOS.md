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

## ~~Bundle 0 — Repo Sync (prerequisite for everything)~~

> ~~Get the GitHub repo into a known good state so the agent always works from ground truth.~~

- [x] ~~**T0.1 — Commit RLS migration**~~
  ~~Write and commit `supabase/migrations/20260304000000_rls_policies.sql` capturing the four granular RLS policies currently applied server-side but missing from the repo. Drop the old `authenticated_full_access` catch-all if present. See `docs/ARCHITECTURE.md` for exact policy definitions. Push to `main`.~~

- [x] ~~**T0.2 — Upgrade edge function model to Sonnet**~~
  ~~In `supabase/functions/tag-image/index.ts`, replace `claude-haiku-4-5-20251001` with `claude-sonnet-4-5-20250929`. Commit and push. Verify the function still deploys cleanly.~~

---

## Bundle 1 — Edit Existing Library Entries (P1)

> `SignCard.jsx` has an `onUpdate` callback wired up but no edit UI. Editing tags on approved signs is needed for data quality.

- [x] ~~**T1.1 — Spec: Edit tags on library entries**~~
  ~~PM Agent writes spec covering: inline edit mode on SignCard, which fields are editable (all tag fields + owner_notes + quality_rating + is_featured + is_approved), save/cancel flow, optimistic UI update, Supabase `update()` call with `updated_at` timestamp.~~
  Spec: `specs/bundle-1-edit-library-entries.md`

- [x] ~~**T1.2 — Implement edit UI**~~
  ~~Coding Agent implements per spec. Fields: all tag columns from schema. Requires updating `SignCard.jsx`, `LibraryPage.jsx`, and potentially extracting an `EditModal.jsx` or `EditDrawer.jsx` component.~~
  Commit: `signforge-admin@f8350dd`

- [x] ~~**T1.3 — QA edit flow**~~
  ~~Test: edit a tag, save, verify update reflected in library. Test: cancel edit, verify no change. Test: edit `is_approved` and `is_featured` toggles.~~
  All 6 checks passed. `quality_rating` stores as integer correctly. Minor: Save button requires precise click target — sticky footer clips near modal edge. Non-blocking.

---

## Bundle 1.5 — Edit Modal Polish (P2)

- [ ] **T1.4 — Fix Save button click target in EditModal**
  The sticky footer in `EditModal.jsx` clips near the bottom edge of the modal scroll container, making the Save button require a very precise click. Add `min-h-[52px]` or padding to the footer, or increase the button's hit area.

- [x] ~~**T1.5 — Verify `.env` is gitignored in signforge-admin**~~
  ~~`.env` was created locally during QA setup. Confirm it's listed in `.gitignore` so it can't be accidentally committed.~~
  Confirmed — `.env` is in `.gitignore` (line 14).

---

## Bundle 2 — Upload Pipeline (P1)

> Bulk upload is the next major workflow feature. Absorbing upload quality improvements here since they touch the same files and are more impactful at bulk scale.

- [ ] **T2.1 — Spec: Bulk upload queue**
  PM Agent writes spec covering: multi-file selection in `UploadZone.jsx`, queue state management (pending / processing / done / error per file), sequential calls to `tag-image` edge function, progress display, error handling per file without aborting the queue.

- [ ] **T2.2 — Spec: Upload pipeline quality**
  PM Agent writes spec covering: (a) persist in-progress tag review state to `sessionStorage` in `TagReviewForm.jsx` and restore on mount — prevents losing a 30-60s Claude Vision result on accidental refresh; (b) resize images client-side to max 2048px before upload to Supabase Storage; (c) compress images client-side (~80% quality) using `canvas` or `browser-image-compression` before upload — reduces storage cost and library load time.

- [ ] **T2.3 — Implement bulk upload + pipeline quality**
  Coding Agent implements per specs. Files: `UploadZone.jsx`, `UploadPage.jsx`, `TagReviewForm.jsx`, `supabase/functions/tag-image/index.ts` (resize before base64 encode for Claude Vision call).

- [ ] **T2.4 — QA upload pipeline**
  Test: upload 5 images, verify all tagged sequentially. Test: one bad file in queue, verify others complete. Test: progress UI updates correctly. Test: refresh mid-review, verify tag state restored. Test: upload a large image, verify it is resized/compressed before storage.

---

## Bundle 3 — Library UX Improvements (P2)

- [ ] **T3.1 — Pagination**
  `LibraryPage.jsx` fetches with a `limit` but no pagination UI. Add "Load more" or page controls. Keep current filter state across pages.

- [ ] **T3.2 — Featured flag toggle**
  `is_featured` exists in schema but no UI to toggle it. Add toggle to `SignCard.jsx` (visible in library, behind `is_approved` gate or always visible — PM to decide in spec).

- [ ] **T3.3 — Search/filter debouncing**
  Every keystroke in the library search input and filter selects triggers a Supabase query. Add a ~300ms debounce in `LibraryPage.jsx` to reduce unnecessary API calls.

- [ ] **T3.4 — Lazy-load library images**
  `SignCard.jsx` uses plain `<img>` tags. Add `loading="lazy"` to prevent the browser from fetching all images at once on library load.

---

## Bundle 5 — Polish (P2)

- [ ] **T5.1 — Error message granularity**
  Map Supabase error codes and edge function failure modes to specific, actionable messages (e.g. "Image too large for analysis", "Session expired — please log in again", "Storage quota exceeded"). Currently most failure paths show generic messages.

- [ ] **T5.2 — Mobile responsiveness**
  Library grid is fixed 3-column with no breakpoints. Tag review form's two-column layout breaks on small screens. Add Tailwind responsive prefixes (`sm:`, `md:`) across `LibraryPage.jsx`, `TagReviewForm.jsx`, and `SignCard.jsx`.

---

## Bundle 6 — Vector Search / Semantic Similarity (P3)

> Higher effort. Needs pgvector enabled in Supabase + embedding generation in the edge function.

- [ ] **T6.1 — Spec: Embedding pipeline**
  PM Agent scopes enabling pgvector, adding embedding generation to `tag-image` (use `ai_description` text as input), storing in `embedding` column, building a similarity search endpoint.

- [ ] **T6.2 — Implement embedding pipeline**

- [ ] **T6.3 — Style matching UI**
  Allow users to input a text description and find similar signs by vector similarity. This is the primary end-goal use case.

---

## Backlog (P3 — no bundle assigned)

- [ ] **Global state layer** — Replace prop-drilling with React Context + `useReducer` for auth state and library filters. Not urgent until more features are added.
- [ ] **Analytics dashboard** — Basic aggregate queries (tags-per-category, upload frequency, average quality rating, top aesthetics). New page in the app.
- [ ] **Test suite** — Integration tests for critical paths (upload flow, tag save, library filtering) using Vitest + React Testing Library.

---

## Completed

### Bundle 1 — Edit Existing Library Entries
- [x] T1.1 — Spec written (`specs/bundle-1-edit-library-entries.md`)
- [x] T1.2 — EditModal + SignCard implemented, commit `signforge-admin@f8350dd`
- [x] T1.3 — QA passed (all 6 checks). Minor: Save button click target small at modal edge — non-blocking.

### Bundle 0 — Repo Sync
- [x] T0.1 — RLS migration committed (`supabase/migrations/20260304000000_rls_policies.sql`)
- [x] T0.2 — Edge function upgraded to `claude-sonnet-4-5-20250929`
