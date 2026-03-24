---
description: Template for generating agent context packets during sprint planning
---

# Agent Context Packet Template

Use this template when assigning an issue to a Worker Agent during sprint planning.
Fill in every section — agents should need zero clarification to start.

---

## Context Packet — Agent-<N>

### Assignment
- **Issue**: #<NUMBER> — <TITLE>
- **Branch**: `feat/<NUMBER>-<slug>`
- **Domain**: `domain:<label>`
- **Priority**: P<X>-<priority>

### Scope

**Files you MAY edit:**
- `path/to/file1.ext` (reason for edit)
- `path/to/file2.ext` [NEW] (new file description)

**Files you MUST NOT edit:**
- `path/to/forbidden1.ext` (owned by Agent-<X>)
- `path/to/forbidden2.ext` (shared file — wait for merge)

### API Contracts

```language
// Define the exact interfaces this agent must implement/export.
// Include function signatures, data models, and expected behavior.
```

### Expected Behavior

- Bullet list of what the feature should do
- Include edge cases and error handling
- Reference acceptance criteria from the issue

### Dependencies

- **Blocked by**: #<N> (reason) — or "None"
- **Blocks**: #<N> (what depends on this work) — or "None"

### Recommended Workflows

- `[/unit-test]` (for generating requirement-driven tests)
- `[/refactor]` (when refactoring complex logic)
- `[/code-review]` (for self-review before PR)

### Coordination Notes

⚠️ Note any shared files, merge order, or cross-agent dependencies here.

### Workflow

```
git fetch origin
git worktree add ../.worktrees/feat/<NUMBER>-<slug> -b feat/<NUMBER>-<slug> origin/main
cd ../.worktrees/feat/<NUMBER>-<slug>
```

Follow `.agent/workflows/worker-agent.md` for full lifecycle.

### Rules Reminder

- Only edit files in your permitted scope
- Commit format: `feat(#<N>): <description>`
- Post progress updates on the issue after each milestone
- Update labels when changing status
