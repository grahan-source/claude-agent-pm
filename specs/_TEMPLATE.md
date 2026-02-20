# Batch N: [Title] (Px)

| Field        | Value                         |
|--------------|-------------------------------|
| **Batch**    | N                             |
| **Repo**     | `repo-name`                   |
| **Priority** | Px — [brief justification]    |
| **Status**   | Ready                         |
| **Files**    | `path/to/file1.ext`, `path/to/file2.ext` |

---

## Problem

[2-3 sentences: What's wrong or what's needed. Why this matters.]

## Architecture

[How the solution fits into the existing system. What approach was chosen and why.]

---

## Task 1: [Title]

**File:** `path/to/file.ext`

[Description of what to change. Include before/after code snippets when possible.]

### Current code (around line N):

```language
// existing code that will be modified
```

### New code:

```language
// what it should become
```

### Important notes:
- [Gotchas, edge cases, things the agent might get wrong]

---

## Task 2: [Title]

**File:** `path/to/file.ext`

[Same structure as Task 1]

---

## What NOT to Touch

- `path/to/unrelated-file.ext` — looks related but should be left alone because [reason]
- [Other files that might be tempting to modify]

---

## Verification Checklist

- [ ] `file1.ext` has [expected change]
- [ ] `file2.ext` has [expected change]
- [ ] [Lint/syntax check command] passes
- [ ] [Functional verification command] returns expected result
- [ ] Only N files changed: [list them]

### Verification commands:

```bash
# Syntax check
[language-specific lint command]

# Functional check
curl -s 'https://staging-url/affected-page/' | grep -c 'expected-element'

# Negative check (ensure nothing broke)
curl -s 'https://staging-url/unrelated-page/' | grep -c 'still-present'
```

---

## Commit Message

```
[Short description of what this batch does]

[Optional longer description with context]
```

---

## Risks & Mitigations

- **[Risk]**: [What could go wrong] — **Mitigation**: [How the spec prevents it]
