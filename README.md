# Claude Agent PM

A production-tested framework for managing software projects with multiple AI coding agents using [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

This isn't a toy. It's the system used to build, ship, and maintain a multi-repo project with 9 repositories, a data warehouse, Cloud Functions, and CI/CD pipelines — all coordinated by AI agents with a human product owner making decisions.

---

## What This Is

A set of agent role definitions, operating principles, task management conventions, spec templates, and workflow patterns that let you:

- **Run a PM agent** that scopes work, writes specs, triages bugs, and coordinates execution
- **Dispatch coding agents** that follow specs precisely, commit, push, and hand off to QA
- **Run QA agents** that visually inspect, regression test, and report findings
- **Track all work** in a single `TODOS.md` with priorities, bundles, and file checkout
- **Parallelize safely** across multiple repos and agents using a file-level lock table
- **Maintain continuity** across sessions with living architecture docs and regression checklists

The key insight: **AI agent time is cheap; ongoing complexity is expensive.** The framework optimizes for clear specs, observable outcomes, and minimal coordination overhead.

---

## How It Works

```
You report issue or request a feature
    |
    v
PM Agent investigates (reads code, checks data, researches)
    |
    v
PM Agent writes spec (specs/batch-N-*.md)
    |
    v
PM Agent dispatches to Coding Agent with handoff format
    |
    v
Coding Agent executes spec, commits, pushes, writes QA handoff
    |
    v
CI/CD pipeline runs (deploy + cache purge + smoke tests)
    |
    v
QA Agent tests: spec compliance, visual, regression, security, perf
    |
    v
PM Agent reviews QA report, updates TODOS.md, marks done
```

### Agent Roles

| Agent | Does | Doesn't |
|-------|------|---------|
| **PM Agent** | Scopes work, writes specs, triages, verifies deploys, manages TODOS.md | Write application code |
| **Coding Agent** | Follows specs precisely, commits, pushes, writes QA handoff | Make architecture decisions, deviate from spec |
| **QA Agent** | Visual testing, regression, security review, performance checks | Fix issues (only reports them) |
| **Intake Agent** | Logs bugs/requests, triages priority, formats items | Make decisions, write specs, execute |

### Task Management

Work is organized into **bundles** — related tasks that ship together.

```
Bundle = one spec = one agent execution = one QA pass = one deploy
```

Each bundle has:
- A priority (P0-P3)
- A repo assignment
- A spec link
- A file manifest (for parallel safety)
- A status (Needs Spec -> Ready -> In Progress -> Code Complete -> Deployed -> QA -> Live)

### Parallel Safety (File Checkout Protocol)

Multiple coding agents can run simultaneously when their file sets don't overlap.

The Active Work table in TODOS.md tracks what's checked out:

```
| Bundle | Repo       | Files Checked Out               | Started    |
|--------|------------|---------------------------------|------------|
| B12    | app-theme  | inc/auth.php, assets/js/auth.js | 2026-02-20 |
| B13    | app-plugin | inc/api.php                     | 2026-02-20 |
```

Rules:
1. Cross-repo work is always safe to parallelize
2. Same-repo work requires disjoint file sets
3. Shared files (e.g., `functions.php`, `style.css`) serialize work
4. PM agent checks the Active Work table before every dispatch
5. Coding agents cannot edit files outside their declared file set

This solves a problem that even Anthropic's official Agent Teams feature doesn't address — file-level conflict prevention across parallel agents.

---

## File Structure

```
your-project/
├── CLAUDE.md              # Operating principles (highest priority)
├── TODOS.md               # Task tracking, bundles, priorities
├── PM-AGENT.md            # PM agent role specification
├── CODING-AGENT.md        # Coding agent role specification
├── QA-AGENT.md            # QA agent role specification
├── INTAKE-AGENT.md        # Intake/triage agent role specification
├── specs/
│   ├── _TEMPLATE.md       # Blank spec template
│   ├── example-batch-1-user-auth.md  # Example filled spec
│   └── qa/
│       └── _TEMPLATE.md   # QA handoff template
├── .github/
│   └── workflows/
│       └── deploy.yml     # Example deploy + cache purge + smoke test
└── docs/
    └── ARCHITECTURE.md    # Living architecture reference template
```

---

## Getting Started

### 1. Fork or clone this repo

Use it as the "PM repo" — the coordination hub. Your application code lives in separate repos.

### 2. Customize CLAUDE.md

The operating principles are opinionated but general-purpose. Review them and adjust to your project's values. These have the highest priority — they override individual task instructions when there's a conflict.

### 3. Set up TODOS.md

Start with the template. Add your initial tasks. The format matters — it's designed for AI agents to parse and update reliably.

### 4. Fill in docs/ARCHITECTURE.md

This is the living reference for your system. Schemas, pipelines, credentials, repo map. The fastest path for any new session to understand your project.

### 5. Write your first spec

Copy `specs/_TEMPLATE.md`, fill it in for a real task. The more precise you are (file paths, line numbers, code snippets showing before/after), the fewer errors the coding agent makes.

### 6. Run agents via Claude Code

Start Claude Code in your PM repo. The agent reads `CLAUDE.md` automatically.

- **PM session:** Work in the PM repo. Reads code across all repos, writes specs, manages TODOS.md.
- **Coding session:** Start Claude Code in the application repo. Point the agent to `CODING-AGENT.md` and the batch spec.
- **QA session:** Start Claude Code with access to the staging site. Point the agent to `QA-AGENT.md` and the QA handoff.

---

## Design Decisions

### Why separate agents instead of one?

Separation of concerns. The PM agent has broad context but doesn't write code. The coding agent has deep file context but doesn't make architecture decisions. The QA agent tests from the outside like a real user. This prevents scope creep and keeps each session focused.

### Why bundles instead of individual tasks?

Bundles are the unit of deployment. Related changes ship together, get tested together, and roll back together. One bundle = one commit, one pipeline run, one QA pass.

### Why a file checkout table?

When two agents edit the same file simultaneously, one overwrites the other. The checkout table prevents this. It's a lightweight mutex that works because the PM agent is the single coordinator.

### Why specs instead of just telling the agent what to do?

Specs are:
- **Reviewable** before execution starts
- **Reproducible** if the agent fails and needs to retry
- **Auditable** after the fact
- **Reusable** as patterns for similar future work

### Why operating principles in CLAUDE.md?

They're the tiebreaker. When a spec says "add a quick fix" but CLAUDE.md says "fix the system, not the symptom," the principles win. This prevents accumulated tech debt across dozens of agent executions.

### Why TODOS.md instead of GitHub Issues or a task API?

- Zero dependencies — no API calls, no auth tokens
- Version-controlled in git — full history of every change
- Human-readable without tooling
- AI agents can read and write it reliably
- Works offline

The tradeoff is visibility — GitHub Issues are easier for teams to browse. For a solo developer + agents, TODOS.md is simpler and faster.

---

## Prior Art & Influences

This framework was developed independently but shares DNA with several projects:

- **[GitHub Spec Kit](https://github.com/github/spec-kit)** — The "Constitution" concept (project-level principles governing all specs) mirrors our CLAUDE.md approach
- **[CCPM](https://github.com/automazeio/ccpm)** — Claude Code project management using GitHub Issues. Our file checkout protocol is more explicit; their GitHub Issues integration has better multi-user visibility
- **[BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD)** — Analyst -> PM -> Architect -> Implementer pipeline. More ceremony than this system, but good patterns for larger features
- **[Anthropic's C Compiler](https://www.anthropic.com/engineering/building-c-compiler)** — 16 parallel Claude agents, 100K lines of code. Proved that file-based locking via git works at scale with minimal orchestration
- **[MetaGPT](https://github.com/FoundationAgents/MetaGPT)** — Multi-agent software company simulation. Academic/research-oriented but pioneered the "standard operating procedures" concept for agents

Key differences from most frameworks:
1. **File checkout protocol** — Almost no other system has explicit file manifests per work unit
2. **Bundle-based work units** — Most systems track individual tasks, not deployment groups
3. **Decision philosophy in CLAUDE.md** — Most instructions files are coding style guides; ours encodes how to think about problems
4. **Cross-repo coordination** — Most multi-agent systems work within a single repo

---

## Semi-Formal Code Reasoning

All three agent roles include structured reasoning protocols based on [research on agentic code reasoning](https://arxiv.org/abs/2603.01896). The core insight: forcing agents to trace execution paths with concrete inputs catches 10-15% more bugs than unstructured "looks right" review.

### Coding Agent: Self-Verification Certificate

Before committing, the coding agent fills out a reasoning certificate for each task — stating premises from the spec, tracing execution with concrete inputs (happy path, edge case, error path), and providing a formal conclusion. This prevents "pattern-matching" pushes where the code looks right but breaks under different inputs.

### QA Agent: Code Reasoning (Diff Analysis)

The QA agent reads the actual diff and answers structured questions: What data flows through this code? What are all the branches? Does the change match the spec's intent semantically? This layer catches bugs that visual testing and curl checks miss — issues that only surface with null values, empty arrays, or unexpected types.

### PM Agent: Fault Localization Protocol

When investigating bugs, the PM agent follows a phased protocol: symptom characterization → test semantics → code path trace → divergence analysis → formal prediction. This prevents fixating on the crash site and missing the actual root cause, especially for indirection bugs and multi-file issues.

---

## Lessons Learned (from production use)

1. **Exact code snippets in specs > descriptions of what to change.** Ambiguity causes wrong implementations.
2. **Visual QA catches what CLI checks miss.** `curl | grep` can show correct HTML while the page looks broken.
3. **Completed items are a regression checklist.** Don't delete them — they document what should still work.
4. **Agent time is cheap; your review time is expensive.** Write thorough specs upfront rather than iterating through bad implementations.
5. **Scope creep kills agent sessions.** If the agent discovers something adjacent, it should stop and report, not improvise.
6. **Living architecture docs eliminate re-investigation.** Every session starts fresh — good docs are the fastest onramp.
7. **Agents suffer "time blindness."** They'll happily spend hours running tests instead of making progress. Set expectations for speed in your specs.
8. **Observability beats speed.** A slow system you can see is better than a fast system you can't explain. Every failure at scale traces back to insufficient observability.
9. **Structured reasoning > unstructured review.** Semi-formal verification certificates catch semantic bugs that "does it look right?" misses. The 2.8x step overhead is irrelevant — agent time is cheap, bugs that reach production aren't.

---

## License

MIT
