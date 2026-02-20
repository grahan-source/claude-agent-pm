<!--
TODOS.md — Task tracking for [Your Project]

Organization:
  Tasks are grouped into BUNDLES — related work that ships together.
  Each bundle = one agent execution = one QA pass = one deploy.
  Bundles are ordered by execution priority (highest first).

Intake:
  New bugs/requests tagged [NEEDS TRIAGE] with a suggested Px (P0-P3).
  PM agent assigns to an existing bundle or creates a new one.
    P0 = Broken / blocking users now
    P1 = Must fix before launch
    P2 = Post-launch improvement
    P3 = Backlog

Bundle format:
  **Repo:** where the code lives
  **Spec:** link to spec (or "Needs spec")
  **Status:** one of the stages below
  **Files:** every file the bundle will create or modify

Status stages (in pipeline order):
  Needs Spec     — not yet planned
  Ready          — spec'd, waiting for dispatch to coding agent
  In Progress    — coding agent actively working
  Code Complete  — code written locally, NOT committed/pushed/deployed
  Deployed       — committed, pushed, CI/CD deployed to server
  QA             — PM verifying on live site
  Live           — verified working in production, stable
  Blocked        — can't proceed (with reason)

  A bundle can have mixed stages across sub-items.
  Use the LOWEST stage of any incomplete item as the bundle status.

Line-item markers:
  - [ ] not started
  - [x] ~~strikethrough~~ — deployed + verified on live site
  Items that are coded but not deployed: uncheck and mark "Code complete (uncommitted)"

File checkout (parallel agents):
  The "Active Work" table tracks which bundles are in progress.
  Before dispatching a bundle, PM checks for file overlaps.
  Two bundles run in parallel ONLY if file sets are disjoint.
  Cross-repo work is always safe — no file conflicts possible.
  Same-repo work requires disjoint file sets.
  High-contention files (functions.php, style.css) serialize work.
  TODOS.md and CLAUDE.md are PM-agent-only — coding agents never write them.

Completion:
  Mark tasks with ~~strikethrough~~ only after deployed + verified on live site.
  Completed bundles (all items Live) move to the Completed section (regression checklist).
  "Code Complete" is NOT "Done" — code must be committed, deployed, and QA'd.
-->

## Active Work

<!-- PM agent updates this table when dispatching bundles and clearing QA. -->
<!-- Coding agents: check this table before starting. If your files overlap, STOP. -->

| Bundle | Repo | Files Checked Out | Started |
|--------|------|-------------------|---------|
| (none) | — | — | — |

---

## Active Bundles

### [P1] Example: User Authentication
**Repo:** app-theme | **Spec:** [batch-1](specs/example-batch-1-user-auth.md) | **Status:** Ready
**Files:** `inc/auth.php` (new), `inc/template-tags.php`, `assets/js/auth.js` (new)

- [ ] **Login form component** — Create login/logout form partial. Uses existing `.form-field` BEM pattern. Nonce protected.
- [ ] **Session management** — WordPress native sessions via `wp_set_auth_cookie()`. Redirect after login to referring page.
- [ ] **Protected content shortcode** — `[members_only]...[/members_only]` shortcode for gated content.

**QA:** Login → redirect works. Logout clears session. Protected content hidden for logged-out users. Nonce present on form. No XSS in error messages.

---

### [P2] Example: Performance Optimization
**Repo:** app-theme | **Spec:** Needs spec | **Status:** Needs Spec

- [ ] **Critical CSS extraction** — Inline above-fold CSS, defer the rest
- [ ] **Image lazy loading audit** — Verify all below-fold images have `loading="lazy"`
- [ ] **JS bundle splitting** — Split admin JS from frontend JS

**QA:** LCP < 2.5s. No layout shifts from lazy images. Admin pages still functional.

---

### [P3] Cleanup & Housekeeping
**Repo:** various | **Spec:** None | **Status:** Ready

- [ ] **Remove dead CSS** — Audit with PurgeCSS, remove unused rules
- [ ] **Update dependencies** — Review and update npm/composer packages

---

## User Todos

Things that need human action — not code tasks.

### Decisions Needed
- [ ] Choose email provider for transactional emails (SendGrid vs SES vs Postmark)
- [ ] Confirm content migration strategy with team

### Access / Credentials
- [ ] Provide API keys for third-party integrations

---

## Completed (Regression Checklist)

<!-- Items here are Live — deployed, verified, in production. -->
<!-- This section IS the regression checklist. Every item documents something that should still work. -->
<!-- Format: ~~**Title**~~: What was done — commit ref. -->

_(No completed items yet — they'll accumulate here as bundles ship.)_
