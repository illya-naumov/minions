---
description: Bootstrap a session as a Sprint Worker Agent
---

# Start Sprint Agent

// turbo-all

## Overview

Use this workflow to bootstrap a new session as a worker agent for the current sprint.
Each agent works through a **queue of issues** from the GitHub Project board assigned to its unique `Worker-ID`.
Each issue has a **context packet** posted as a comment, and **SDD artifacts** (`spec.md`, `design.md`, `task.md`) which guide execution.

---

## 0. Get Your Queue

Query the GitHub Project board for your assigned issues:

```
gh project item-list <PROJECT_NUMBER> --owner <OWNER> --format json
```

Filter the output for items where the `Agent` field matches your assigned `Worker-ID` (e.g., `Worker-A`).
These are your issues, **ordered by priority**.

## 1. Read Your Context Packet & Artifacts

For each issue in your queue:
1. Read the context packet posted as a comment on the issue.
2. Read the Spec-Driven Development artifacts linked or attached to the issue:
   - `spec.md` (What are we building and what is success?)
   - `design.md` (How are we building it technically?)
   - `task.md` (What are the atomic chunks of work?)

## 2. Resume Check

Before creating a fresh worktree, check if progress was already made. Look for remote branches matching your issue feature branch. If resuming, read the latest changes in the worktree and update your internal state.

## 3. Create Your Isolated Worktree

**DO NOT use `git checkout`**. Use isolated worktrees:

```
git fetch origin
git worktree add ../.worktrees/<your-branch-name> -b <your-branch-name> origin/main
cd ../.worktrees/<your-branch-name>
```

**CRITICAL**: ALL file edits and commands MUST run from your worktree path.

## 4. Post Status: Work Started

Post a comment on the issue:
```bash
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "⏳ <WORKER_ID> has started work. Branch: \`<branch>\`."
```

## 5. Execute in Atomic Chunks

Follow the plan detailed in `task.md` strictly:
- Build the feature in small, atomic chunks.
- Only edit permitted files in your domain.
- Use the specified API contracts from `design.md`.

Keep the `task.md` updated as you complete each chunk.

## 6. Verification

Verify that your execution matches the plan:
- Run unit tests and any other applicable tests.
- Cross-reference with the KPIs and User Journeys defined in `spec.md`.

## 7. Reconcile (Drift Detection)

Before you are done, perform drift detection:
- Did you deviate from the original `design.md` architecture or API contracts during execution?
- If so, **update `design.md` and `spec.md`** to reflect the real-world code.
- Ensure `task.md` is fully checked off.
- The goal is to align the real-world code to the ideal-world spec.

## 8. Push & PR

```
git add .
git commit -m "feat(#<ISSUE_NUMBER>): <description>"
git pull --rebase origin main
git push origin <your-branch>
gh pr create --fill --base main --repo <OWNER>/<REPO>
```

## 9. Report Completion

Post a comment on the issue:
```bash
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "📋 **PR #<PR> submitted.** Summary: <1-line>. All drifts reconciled."
```

## 10. Next Issue in Queue

After completing an issue, move to the next issue assigned to your `Worker-ID` in the Project board. Repeat the cycle from Step 1.

Once your full queue is empty, proceed to Step 11.

## 11. Retrospective (Self-Improvement)

After completing your **entire queue**, analyze this workflow run and propose improvements. This step is mandatory — do not skip it even if everything went smoothly.

### a) Analyze the Run

Review your execution and categorize issues into this table:

| Category | What to Look For |
|---|---|
| **Command failures** | Any CLI command that errored, wasn't available, or needed different flags for this OS/shell |
| **Missing steps** | Prerequisites you had to figure out that weren't documented (e.g., `npm install` before build) |
| **Ambiguous instructions** | Steps where you had to guess intent, made wrong assumptions, or wasted effort |
| **Context packet gaps** | Information missing from the context packet that you needed (API contracts, file paths, constraints) |
| **Domain violations** | Files you were tempted to edit outside your permitted domain, or files you edited that you shouldn't have |
| **Encoding / platform issues** | PowerShell quirks, path format problems, line ending mismatches, UTF-8 issues |
| **Workflow gaps** | Situations that arose (e.g., merge conflicts, CI failures, missing deps) that the workflow didn't cover |

### b) Generate Retrospective Report

Produce a structured report:

```markdown
## 🔄 Workflow Retrospective — Agent-<N>

### Run Summary
- **Issues completed:** #<list>
- **Duration:** ~<time>
- **Overall result:** <success / partial / blocked>

### Issues Encountered
| # | Category | Severity | Description |
|---|----------|----------|-------------|
| 1 | <category> | 🔴/🟡/🟢 | <what happened> |

### Proposed Workflow Edits
> ⚠️ These edits require user approval before applying.

#### Proposal 1: <title>
**File:** `.agent/workflows/worker-agent.md`
**Reason:** <why this change would help>
```diff
-<old line>
+<new line>
```

### No Issues Found
<If nothing went wrong, state: "No issues encountered. Workflow performed as documented.">
```

### c) Present to User

Surface the retrospective report to the user via `notify_user` with `BlockedOnUser: true`. Include the workflow file in `PathsToReview` **only if** you have proposed edits.

**Rules:**
- You MUST NOT auto-edit any workflow `.md` file. All changes require explicit user approval.
- Proposals should be minimal and targeted — do not rewrite entire sections.
- If no issues were found, still report the clean run (valuable signal) but set `BlockedOnUser: false`.
