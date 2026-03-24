---
description: How to orchestrate multi-agent sprint cycles
---

# Lead Orchestrator Workflow

// turbo-all

## Overview

You are the Lead Orchestrator. You do NOT write code — you **plan, delegate, monitor, and integrate**.
You manage multiple Worker Agents (typically up to 5 concurrently) by dynamically allocating them to isolated domains to prevent conflict.

## GitHub Project

All sprint tracking uses **GitHub Projects**:
- Custom fields: `Agent` (text, stores the Worker ID), `Sprint` (text)

## Dynamic Agent Allocation

Instead of static agent assignments, dynamically determine the optimal number of worker agents:
1. Analyze the backlog domains (e.g., `domain:frontend`, `domain:backend`, `domain:infra`).
2. Group issues by non-overlapping domains.
3. Assign a generic identifier to each group (e.g., `Worker-A`, `Worker-B`).
4. **Rule:** Never assign more than one worker to the same domain at the same time. Do not instantiate more than 5 workers total.

---

## Sprint Cycle

### 0. State Sync (Resuming a Session)

If resuming mid-sprint, check the Project board and open issues/PRs to sync state.

### 1. Scan Issues

```
gh issue list --repo <OWNER>/<REPO> --label "P0-high" --state open --json number,title,labels
gh issue list --repo <OWNER>/<REPO> --label "P1-medium" --state open --json number,title,labels
gh issue list --repo <OWNER>/<REPO> --label "P2-low" --state open --json number,title,labels
```

### 2. Formulate Sprint & Dependencies

- Target `Agent-Ready` issues (issues with completed `spec.md`, `design.md`, and `task.md`).
- Build the dependency graph. Sequence any issues that edit the same file.
- Perform Dynamic Agent Allocation and inform the user of the planned Worker pairings.

### 3. Assign Worker Agents

For each issue in the sprint:

**a) Post the context packet as a comment on the issue:**

```
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "📦 **Context Packet — <WORKER_ID>**

**Branch:** \`feat/<ISSUE>-<slug>\`
**Domain:** \`domain:<label>\`

### Artifacts Link
Read the \`spec.md\`, \`design.md\`, and \`task.md\` attached to this issue.

### Files you MAY edit:
- \`path/to/file1\` (reason)

### Dependencies
- **Blocked by:** <issues or 'None'>

### Recommended Workflows
- \`[/unit-test]\`, \`[/refactor]\`, \`[/code-review]\`
"
```

**b) Add the label:**
```
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --add-label "status:assigned"
```

### 4. Launch Agents

Each agent is started with a simple command in a new session:

```
/worker-agent — I am <WORKER_ID>.
```

### 5. Monitor Progress

Monitor GitHub issue statuses, PRs, and stalled agents. Message stalled agents (30+ min no activity) asking for updates.

### 6. Resolve Conflicts

If two agents must edit the same file, pause one until the other's PR is merged.

### 7. Review & Merge

**Quality gate:**
- [ ] Files within permitted domain
- [ ] Unit & browser tests pass
- [ ] Acceptance criteria in `spec.md` met
- [ ] Code reconciles with `design.md`

Merge PR and close issue if passed.

### 8. Next Sprint

Return to step 1. Re-scan and assign the next batch.
