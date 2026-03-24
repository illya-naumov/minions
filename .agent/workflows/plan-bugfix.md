---
description: End-to-end bug fix planning — repro, root cause, fix plan, tasks, and staging
---

# Bug Fix Planning Workflow

// turbo-all

Take a bug report from repro to an Agent-Ready GitHub issue — no code touched.

> **Note on Multi-Agent Sprints:** Execution happens through `[/worker-agent]`. This workflow handles the **Investigation** and **Plan** phases of Spec-Driven Development for bug fixes.

## Steps

### 1. Build the Bug Report

Collaborate with the user to build a structured bug report:
1. **Symptom:** What is the user-visible behavior?
2. **Expected behavior:** What *should* happen?
3. **Repro steps:** Minimum steps to reproduce.
4. **Environment:** OS, browser, runtime version, deploy target.
5. **Evidence:** Error messages, stack traces, screenshots, logs.

### 2. Root Cause Analysis

Systematically investigate the codebase to find the root cause:
1. **Locate the symptom:** Trace the error from the user-visible behavior to the code path.
2. **Identify the trigger:** What input, state, or timing causes the bug?
3. **Find the root cause:** What is the underlying code defect?
4. **Assess blast radius:** What else could be affected?

Document findings as:
- **Root cause:** 1-2 sentence summary
- **Affected files:** List with line references
- **Blast radius:** Other features or paths impacted

### 3. Generate Fix Plan

Once the root cause is confirmed, generate a targeted fix plan:
- **Fix description:** What code changes are needed?
- **Regression tests:** What tests should be added to prevent recurrence?
- **Rollback strategy:** If the fix breaks something else, how do we revert?
- **Task list:** Atomic chunks of work for the worker agent.

Present the fix plan to the user for review. Iterate until approved.

### 4. Stage Issue (Agent-Ready)

Once the user approves the fix plan:
1. Consolidate the bug report, root cause analysis, and fix plan into the GitHub Issue body.
2. Add relevant metadata tags:
   - The appropriate `domain:` label (e.g., `domain:frontend`, `domain:backend`).
   - Priority labels (e.g., `P0-high` for production bugs, `P1-medium` for non-critical).
   - Bug label (e.g., `bug`).
3. Mark the issue as **"Agent-Ready"**.

The bug fix is fully planned! The Lead Orchestrator will assign this issue to a worker agent via `[/orchestrate-sprint]`.

### 5. Retrospective (Self-Improvement)

After the bug fix issue is staged, analyze this workflow run and propose improvements. This step is mandatory.

#### a) Analyze the Run

| Category | What to Look For |
|---|---|
| **Repro quality** | Were the initial repro steps sufficient, or did you need multiple iterations to reproduce? |
| **Root cause accuracy** | Did your first diagnosis hold, or did you have to revise after deeper investigation? |
| **Command failures** | Did any investigation commands (`grep`, `git log`, test runners) fail or need adaptation for this OS/shell? |
| **Ambiguous steps** | Were any workflow steps unclear about what to do (e.g., how deep to investigate vs. when to stop)? |
| **Missing context** | Did you need information not available in the issue or codebase (e.g., deployment logs, user reports)? |
| **Redundant steps** | Were any steps unnecessary or could be combined for efficiency? |

#### b) Generate Retrospective Report

```markdown
## 🔄 Workflow Retrospective — Bug Fix Planning

### Run Summary
- **Bug:** #<issue> — <title>
- **Root cause found:** yes/no
- **Iterations needed:** <count>

### Issues Encountered
| # | Category | Severity | Description |
|---|----------|----------|-------------|
| 1 | <category> | 🔴/🟡/🟢 | <what happened> |

### Proposed Workflow Edits
> ⚠️ These edits require user approval before applying.

#### Proposal 1: <title>
**File:** `.agent/workflows/plan-bugfix.md`
**Reason:** <why this change would help>
```diff
-<old line>
+<new line>
```

### No Issues Found
<If nothing went wrong, state: "No issues encountered. Workflow performed as documented.">
```

#### c) Present to User

Surface the retrospective report to the user via `notify_user` with `BlockedOnUser: true`. Include the workflow file in `PathsToReview` **only if** you have proposed edits.

**Rules:**
- You MUST NOT auto-edit any workflow `.md` file. All changes require explicit user approval.
- Proposals should be minimal and targeted — do not rewrite entire sections.
- If no issues were found, still report the clean run but set `BlockedOnUser: false`.
