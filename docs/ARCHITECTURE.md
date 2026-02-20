# Architecture Reference

> Living document. Update after any session that changes how components connect,
> adds new pipeline steps, or modifies data flow.
>
> This is the fastest path for the next session to understand the system.

---

## Overview

[2-3 sentences: What this project is, who it serves, what the primary goals are.]

---

## Architecture Diagram

```
DATA SOURCES                     YOUR INFRASTRUCTURE
─────────────                    ────────────────────

[Source 1]                       [Database / Warehouse]
  │                                ├── table_1
  │ [sync method]                  ├── table_2
  ▼                                └── table_3
┌──────────┐
│  [Store]  │◄── [Pipeline]
└──────────┘
  ▲
  │
[Source 2]   [Source 3]
```

---

## Current State

### What's Working

| Component | Status | Details |
|-----------|--------|---------|
| [Component 1] | **Live** | [Brief description] |
| [Component 2] | **Live** | [Brief description] |

### What's Planned

| Component | Priority | Notes |
|-----------|----------|-------|
| [Planned 1] | Next | [Brief description] |
| [Planned 2] | Phase 2 | [Brief description] |

---

## Data Inventory

### [Database/Dataset Name]

| Table | Rows | Refresh | Source |
|-------|------|---------|--------|
| `table_name` | ~N | [Schedule] | [Source system] |

### Schemas

#### `table_name`

| Column | Type | Notes |
|--------|------|-------|
| id | INT | Primary key |
| name | STRING | |
| created_at | TIMESTAMP | |

---

## Pipelines / Cloud Functions / Scheduled Jobs

### [Pipeline Name]

| Field | Value |
|-------|-------|
| Entry point | `function_name` |
| Schedule | [cron expression and human-readable] |
| Source | `path/to/source.py` |
| What it does | [1-2 sentences] |
| Dependencies | [packages, APIs, credentials] |

### Job Schedule

| Job | Schedule | Target |
|-----|----------|--------|
| [job-1] | `0 6 * * *` (daily 6am) | [function] |
| [job-2] | `0 5 * * 1` (Mon 5am) | [function] |

---

## Credentials and Access

All credentials stored in `.env` (gitignored).

| Service | Auth method | Key env vars |
|---------|-------------|-------------|
| [Service 1] | API key | `SERVICE_1_API_KEY` |
| [Service 2] | OAuth 2.0 | `SERVICE_2_CLIENT_ID`, `SERVICE_2_CLIENT_SECRET` |
| [Database] | Application credentials | `DB_HOST`, `DB_USER`, `DB_PASSWORD` |

---

## Deploy Pipelines

| Repo | Method | Trigger | Steps |
|------|--------|---------|-------|
| [repo-1] | GitHub Actions → rsync | Push to main | build → deploy → cache purge → smoke test |
| [repo-2] | GitHub Actions → rsync | Push to main | deploy → cache purge |

---

## Repository Map

| Repo | Purpose | Deploy |
|------|---------|--------|
| `pm-repo` | PM repo — specs, scripts, agent defs | N/A (coordination only) |
| `app-theme` | Frontend — templates, CSS, JS | GitHub Actions → rsync |
| `app-plugin` | Backend plugin — API, data models | GitHub Actions → rsync |

---

## Known Limitations

1. **[Limitation 1]**: [Description and workaround]
2. **[Limitation 2]**: [Description and workaround]

---

## Roadmap

### Phase 1: [Title] — DONE
- [x] [Completed item]
- [x] [Completed item]

### Phase 2: [Title] — IN PROGRESS
- [x] [Completed item]
- [ ] [Pending item]

### Phase 3: [Title]
- [ ] [Planned item]
