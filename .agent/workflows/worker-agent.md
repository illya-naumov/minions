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

After completing an issue, move to the next issue assigned to your `Worker-ID` in the Project board. Repeat the cycle.
