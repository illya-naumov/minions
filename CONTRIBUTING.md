# Contributing

Welcome! This project uses **Spec-Driven Development (SDD)** with autonomous AI worker agents. Whether you're a human contributor or an AI agent, this guide covers the rules of engagement.

---

## 1. The SDD Lifecycle

Every feature or fix goes through four phases:

### Phase 1: Spec & Audit
- **Features:** Use `/plan-feature` to build a product spec from raw requirements, then audit for contradictions, missing constraints, and incomplete journeys.
- **Bug Fixes:** Use `/plan-bugfix` to build a structured bug report, perform root cause analysis, and generate a targeted fix plan.

### Phase 2: Plan
- `/plan-feature` generates the technical design (`design.md`) and atomic task list (`task.md`).
- `/plan-bugfix` generates the fix plan with affected files, regression tests, and rollback strategy.
- The user reviews and approves before any code is written.

### Phase 3: Execute
The orchestrator (`/orchestrate-sprint`) assigns `Agent-Ready` GitHub issues to worker agents. Each worker (`/worker-agent`) implements the `task.md` in an isolated `git worktree` branch.

### Phase 4: Reconcile
Before opening a PR, the worker performs drift detection — if the real-world code diverged from the original design, it updates the SDD artifacts to stay in sync.

---

## 2. Required Artifacts

Every feature mandates three artifacts before implementation begins:

| Artifact | Purpose | Template |
|----------|---------|----------|
| `spec.md` | Product criteria, user journeys, constraints, KPIs | `.agent/templates/spec.md` |
| `design.md` | Technical architecture, API contracts, schemas | `.agent/templates/design.md` |
| `task.md` | Granular execution checklist for the worker agent | `.agent/templates/task.md` |

Finalized artifacts are concatenated into the **GitHub Issue body** and the issue is labeled `Agent-Ready`.

---

## 3. Branching & Commits

- **Never commit directly to `main`.** Use isolated worktrees: `feat/<issue>-<slug>`.
- **Rebase before pushing:** `git pull --rebase origin main`.
- **Conventional Commits** are mandatory:

```
<type>(#<issue>): <short description>
```

| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructuring, no behavior change |
| `docs` | Documentation |
| `test` | Tests |
| `chore` | CI, config, dependencies |

---

## 4. Multi-Agent Conflict Avoidance

- Each issue is tagged with a `domain:` label indicating which directories it affects.
- No two agents may operate in the same domain simultaneously.
- Shared files (README, global styles, CI configs) use **append-only** editing with labeled blocks.
- Workers must not merge their own PRs — the orchestrator merges after quality gate review.

---

## 5. Configuration

The `.agent/` directory contains everything the AI needs:

| Resource | Purpose |
|----------|---------|
| `rules.md` | Workspace conventions (design, file org, domains). Auto-loads. |
| `workflows/` | Step-by-step SOPs (see table below). |
| `templates/` | SDD artifact templates (`spec.md`, `design.md`, `task.md`) |
| `skills/` | Curated AI capabilities (security scanning, TDD, UX audits) |

### Core Workflows

| Workflow | Purpose |
|----------|---------|
| `/plan-feature` | Take a raw feature idea from spec → audit → design → agent-ready GitHub issue. |
| `/plan-bugfix` | Take a bug report from repro → root cause → fix plan → agent-ready GitHub issue. |
| `/orchestrate-sprint` | Orchestrate multi-agent sprints: assign, monitor, review, merge, deploy. |
| `/worker-agent` | Bootstrap an agent session to execute its assigned issue queue. |

### Self-Improving Workflows

All core workflows include a mandatory **Retrospective** step that runs after the workflow completes. The agent:
1. Analyzes its run for command failures, missing steps, ambiguous instructions, and workflow gaps.
2. Generates a structured report with proposed `diff` edits to the workflow `.md` file.
3. Surfaces recommendations to the user — **no workflow file is modified without explicit approval**.

---

## 6. Quality Gate

Before merging any PR, the orchestrator verifies:

- [ ] Files are within the agent's permitted domain scope
- [ ] Unit and browser tests pass
- [ ] Acceptance criteria from `spec.md` are met
- [ ] Code reconciles with `design.md` (no unresolved drift)
- [ ] README and docs updated if architecture changed
