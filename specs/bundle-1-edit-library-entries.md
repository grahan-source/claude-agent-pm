# Spec — Bundle 1: Edit Existing Library Entries

**Repo:** `github.com/grahan-source/signforge-admin`
**Branch:** `master`
**Files:** `src/components/SignCard.jsx`, `src/components/EditModal.jsx` (new), `src/pages/LibraryPage.jsx`

---

## Context

`LibraryPage.jsx` already passes `onUpdate={fetchSigns}` to every `SignCard` (line 102), but `SignCard` ignores it — the prop is never destructured. The edit UI just needs to be wired up.

Cards are small (3-column grid), so inline editing is impractical for the number of fields. Use a **modal** instead.

---

## Tasks

### Task 1 — Create `src/components/EditModal.jsx`

New component. Props: `sign` (object), `onSave` (function), `onClose` (function).

**Behavior:**
- Renders a full-screen overlay (`fixed inset-0 bg-black/40 z-50`) with a centered modal panel (max-w-2xl, scrollable)
- Contains a form pre-populated with all editable fields from `sign`
- "Save" button: calls Supabase update, then calls `onSave()`, then closes
- "Cancel" button / clicking overlay: calls `onClose()` with no changes
- Show a saving spinner / disable buttons while the Supabase call is in flight
- Show an inline error message if the update fails (do not throw)

**Editable fields — exact field names from the `signs` schema:**

| Field | Input type | Notes |
|---|---|---|
| `title` | text input | — |
| `product_category` | select | Options: `exterior`, `interior`, `vehicle`, `banners_displays`, `architectural`, `specialty` |
| `product_type` | text input | — |
| `source_type` | text input | — |
| `source_url` | text input | — |
| `aesthetics` | multi-select checkboxes | Options: `minimalist`, `bold`, `vintage`, `modern`, `luxury`, `playful`, `corporate`, `industrial`, `organic`, `artistic` |
| `color_approach` | multi-select checkboxes | Options: `monochromatic`, `complementary`, `analogous`, `triadic`, `neutral`, `vibrant`, `pastel`, `earth_tones` |
| `typography_style` | multi-select checkboxes | Options: `serif`, `sans_serif`, `script`, `display`, `monospace`, `mixed`, `no_text` |
| `complexity` | select | Options: `simple`, `moderate`, `complex` |
| `finish` | multi-select checkboxes | Options: `gloss`, `matte`, `reflective`, `textured`, `backlit`, `dimensional` |
| `lighting` | select | Options: `frontlit`, `backlit`, `halo_lit`, `unlit`, `neon` |
| `installation` | select | Options: `wall_mounted`, `freestanding`, `hanging`, `vehicle`, `ground`, `window` |
| `size_category` | select | Options: `small`, `medium`, `large`, `extra_large` |
| `industries` | multi-select checkboxes | Options: `retail`, `restaurant`, `healthcare`, `automotive`, `real_estate`, `professional_services`, `entertainment`, `education`, `hospitality`, `industrial` |
| `ai_description` | textarea | — |
| `owner_notes` | textarea | — |
| `quality_rating` | number input (1–5) or star picker | — |
| `is_featured` | toggle/checkbox | — |
| `is_approved` | toggle/checkbox | — |

**Do not include:** `id`, `image_path`, `image_url`, `dominant_colors`, `ai_tags_raw`, `embedding`, `created_at`, `created_by`. These are system fields.

**Supabase update call:**
```js
const { error } = await supabase
  .from('signs')
  .update({ ...editedFields, updated_at: new Date().toISOString() })
  .eq('id', sign.id)
```

After a successful update, call `onSave()` (which re-fetches the library) then close the modal.

---

### Task 2 — Update `src/components/SignCard.jsx`

1. Destructure `onUpdate` from props: `export default function SignCard({ sign, onUpdate })`
2. Add local state: `const [editing, setEditing] = useState(false)`
3. Add an "Edit" button — small, appears on card hover or always visible in the bottom-right corner of the content area. Use a pencil icon (`lucide-react` `Pencil` — already a dependency) or plain text "Edit".
4. Clicking "Edit" sets `editing(true)`.
5. When `editing` is true, render `<EditModal>` with:
   - `sign={sign}`
   - `onSave={() => { setEditing(false); onUpdate() }}`
   - `onClose={() => setEditing(false)}`
6. Import `EditModal` from `../components/EditModal` (or `./EditModal` depending on file location).

---

### Task 3 — Verify `src/pages/LibraryPage.jsx`

No changes needed. `onUpdate={fetchSigns}` is already passed on line 102. Confirm this is still true after the SignCard changes — do not modify LibraryPage unless something is broken.

---

## Acceptance Criteria

1. Clicking "Edit" on any library card opens the modal with all fields pre-populated
2. Editing a text field, saving — the library refreshes and shows the updated value
3. Toggling `is_approved` to false — the card disappears from the library (library only shows `is_approved=true`)
4. Toggling `is_featured` to true — the "Featured" badge appears on the card
5. Clicking Cancel (or the overlay) — no changes, modal closes
6. If the Supabase update fails — inline error shown, modal stays open

---

## Commit Message

```
Add edit modal to SignCard — wire up onUpdate callback for library entry editing
```

---

## QA Handoff

After pushing, write `specs/qa/bundle-1-qa-handoff.md` covering the 6 acceptance criteria above.
