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

## Status Labels

| Label | Meaning |
|-------|---------|
| `status:assigned` | Queued for agent, not started |
| `status:in-progress` | Agent actively coding |
| `status:pr-ready` | PR submitted, awaiting review |
| `status:blocked` | Agent stuck, needs help |

---

## Sprint Cycle

### 0. State Sync (Resuming a Session)

If you are starting a new orchestrator session mid-sprint, rebuild your context first:
1. **Check Project Board**: `gh project item-list <PROJECT_NUMBER> --owner <OWNER> --format json`
2. **Check Open Issues**: Verify statuses (`status:in-progress`, `status:pr-ready`, `status:blocked`).
3. **Check Open PRs**: `gh pr list --repo <OWNER>/<REPO> --state open`
4. **Identify Blockers**: Review recent issue comments for stalled agents or CI failures.
5. Once synced, jump to the appropriate step below (e.g., Step 5 for monitoring, Step 7 for review & merge).

### 1. Scan Issues

Determine the sprint type. Ask the user if the sprint should focus on:
- **Features only** — scan `enhancement` label
- **Bug fixes only** — scan `bug` label
- **Both** (default) — scan all `Agent-Ready` issues, agent decides order

```
gh issue list --repo <OWNER>/<REPO> --label "P0-high" --state open --json number,title,labels
gh issue list --repo <OWNER>/<REPO> --label "P1-medium" --state open --json number,title,labels
gh issue list --repo <OWNER>/<REPO> --label "P2-low" --state open --json number,title,labels
```

### 2. Build Dependency Graph

For each issue, check:
- Does this issue require output from another issue first?
- Do two issues edit the same file? If yes, sequence them.

### 3. Assign Agents

For each issue in the sprint, do ALL of the following:

**a) Post the context packet as a comment on the issue:**

```
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "📦 **Context Packet — <WORKER_ID>**

**Branch:** \`feat/<ISSUE>-<slug>\`
**Domain:** \`domain:<label>\`

### Files you MAY edit:
- \`path/to/file1\` (reason)
- \`path/to/file2\` [NEW]

### Files you MUST NOT edit:
- \`path/to/forbidden\` (owned by <OTHER_WORKER>)

### API Contracts
\`\`\`
<interfaces, data models, endpoints>
\`\`\`

### Expected Behavior
- <acceptance criteria bullets>

### Dependencies
- **Blocked by:** <issues or 'None'>
- **Blocks:** <issues or 'None'>

### Recommended Workflows
- `[/unit-test]`, `[/refactor]`, `[/code-review]`

### Coordination
⚠️ <merge order, shared files, cross-agent notes>
"
```

**b) Add the label:**
```
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --add-label "status:assigned"
```

**c) Add to the GitHub Project and set fields:**
```
gh project item-add <PROJECT_NUMBER> --owner <OWNER> --url https://github.com/<OWNER>/<REPO>/issues/<ISSUE_NUMBER>
```
Then set the `Agent` and `Sprint` fields on the project item.

**Multi-issue queues**: An agent can have multiple issues assigned. They work through them sequentially, ordered by priority.

### 4. Launch Agents

Each agent is started with a simple command in a new session:

```
/worker-agent — I am <WORKER_ID>.
```

### 5. Monitor Progress

**GitHub status check:**
```
git fetch origin
git branch -r --list "origin/feat/*"
gh pr list --repo <OWNER>/<REPO> --state open --json number,title,headRefName,createdAt
gh issue list --repo <OWNER>/<REPO> --label "status:in-progress" --state open --json number,title
gh issue list --repo <OWNER>/<REPO> --label "status:pr-ready" --state open --json number,title
gh issue list --repo <OWNER>/<REPO> --label "status:blocked" --state open --json number,title
```

**CI/CD pipeline status (check after each merge):**
```
gh run list --repo <OWNER>/<REPO> --branch main --limit 5 --json databaseId,name,conclusion,createdAt
gh run list --repo <OWNER>/<REPO> --status failure --limit 5 --json databaseId,name,headBranch,createdAt
```

If any deployment failed, immediately stop merging further PRs and triage the failure (see Step 7.5).

**Stalled agent (30+ min no activity):**
```
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "🔍 **Orchestrator check-in:** <WORKER_ID>, please post a progress update."
```

**Dead agent (45+ min no activity):**
```
gh issue comment <ISSUE_NUMBER> --repo <OWNER>/<REPO> --body "💀 **<WORKER_ID> appears dead.** Marking for reassignment."
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --remove-label "status:in-progress" --add-label "status:assigned"
```

### 6. Resolve Conflicts

If two agents must edit the same file:

1. Post hold: `⏸️ **Hold:** Waiting for <WORKER_ID> PR #<X> to merge.`
2. After merge: `▶️ **Unblocked:** PR #<X> merged. Rebase: \`git pull --rebase origin main\``

### 7. Review & Merge

```
gh pr diff <PR_NUMBER> --repo <OWNER>/<REPO> --name-only
```

**Quality gate:**
- [ ] Commits follow conventional format: `<type>(#<issue>): <description>`
- [ ] Files within permitted domain
- [ ] All tests pass
- [ ] Build succeeds
- [ ] Acceptance criteria met
- [ ] Code reconciles with `design.md`

**Pass:**
```
gh pr comment <PR_NUMBER> --repo <OWNER>/<REPO> --body "✅ **Review passed.** Merging."
gh pr merge <PR_NUMBER> --squash --repo <OWNER>/<REPO>
```

**Fail:**
```
gh pr comment <PR_NUMBER> --repo <OWNER>/<REPO> --body "🔄 **Changes requested:** <issues>."
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --remove-label "status:pr-ready" --add-label "status:in-progress"
```

### 7.5. Post-Merge Deployment Gate

After **each** PR merge, verify that all CI/CD pipelines succeed before proceeding to the next merge. Do NOT batch-merge multiple PRs without verifying deployments in between.

**a) Wait for Actions to complete (~30-60s after merge):**
```
gh run list --repo <OWNER>/<REPO> --branch main --limit 5 --json databaseId,name,conclusion,createdAt
```

**b) Check for failures:**
```
gh run list --repo <OWNER>/<REPO> --status failure --limit 5 --json databaseId,name,headBranch,conclusion,createdAt
```

**c) If a deployment failed, get logs and triage:**
```
gh run view <RUN_ID> --repo <OWNER>/<REPO> --log-failed
```

**Triage decision tree:**

| Failure Type | Root Cause | Action |
|---|---|---|
| `stream did not contain valid UTF-8` | Encoding corruption during conflict resolution | Fix encoding: re-read file, strip invalid bytes, write as clean UTF-8 |
| Duplicate content blocks | Sequential rebase appending same content multiple times | Remove duplicate sections, keep first occurrence |
| Missing dependency | New package added but not installed in CI | Check `package.json`/`requirements.txt` has the dep; CI should install |
| Syntax error in build artifact | Broken content from garbled multi-byte chars | Search for corruption markers, fix or regenerate |
| Test failure (assertion mismatch) | UI copy or API response changed but test not updated | Update test assertion to match new behavior |
| `Cannot resolve import` | New dependency added but not in manifest | Install the package, commit manifest + lockfile |

**d) If deployment passes, proceed to next merge.**

**e) If fix required, commit directly to main:**
```
# Fix the issue, verify locally, then:
git add <files>
git commit -m "fix(<scope>): <description of deployment fix>"
git push origin main
```

> ⚠️ **Critical:** When merging multiple PRs that touch the same files, each subsequent PR must be rebased against the updated `main` before merging. Use appropriate merge strategies to avoid content duplication.

### 8. Close Issues

```
gh issue close <ISSUE_NUMBER> --repo <OWNER>/<REPO> --comment "✅ **Completed** in PR #<PR>."
gh issue edit <ISSUE_NUMBER> --repo <OWNER>/<REPO> --remove-label "status:pr-ready"
```

Cleanup:
```
git worktree remove ../.worktrees/<branch>
git branch -d <branch>
```

### 9. Next Sprint

Return to step 1. Re-scan and assign the next batch.

Before starting the next sprint, complete Step 10.

### 10. Sprint Retrospective (Self-Improvement)

After all PRs are merged, issues closed, and deployments verified, analyze this sprint and propose workflow improvements. This step is mandatory.

#### a) Analyze the Sprint

Review the full sprint execution across these categories:

| Category | What to Look For |
|---|---|
| **Merge conflicts** | Which files had conflicts? Were they avoidable with better domain isolation or merge ordering? |
| **CI/CD failures** | What deployment failures occurred? Were they caught quickly or did they cascade? |
| **Agent stalls / deaths** | Did any agents go silent? Was the check-in cadence sufficient? |
| **Context packet quality** | Were agents confused by their packets? Did they edit wrong files or ask clarifying questions? |
| **Shared file collisions** | Did multiple agents edit shared files? How was it handled? |
| **Command failures** | Did any CLI commands fail due to OS, shell, or environment differences? |
| **Workflow gaps** | Did any situation arise that the workflow didn't cover? |
| **Timing issues** | Were polling intervals (merge wait, CI check) too short or too long? |

#### b) Generate Sprint Retrospective Report

```markdown
## 🔄 Sprint Retrospective — Sprint <N>

### Sprint Summary
- **Issues completed:** <count> (<list>)
- **PRs merged:** <count>
- **Deployment failures:** <count> (triaged: <count>)
- **Agent stalls:** <count>

### Issues Encountered
| # | Category | Severity | Description | Impact |
|---|----------|----------|-------------|--------|
| 1 | <category> | 🔴/🟡/🟢 | <what happened> | <time lost / risk> |

### Proposed Workflow Edits
> ⚠️ These edits require user approval before applying.

#### Proposal 1: <title>
**File:** `.agent/workflows/orchestrate-sprint.md`
**Reason:** <why this change would help future sprints>
```diff
-<old line>
+<new line>
```

### No Issues Found
<If nothing went wrong, state: "Clean sprint. Workflow performed as documented.">
```

#### c) Present to User

Surface the retrospective report to the user via `notify_user` with `BlockedOnUser: true`. Include the workflow file in `PathsToReview` **only if** you have proposed edits.

**Rules:**
- You MUST NOT auto-edit any workflow `.md` file. All changes require explicit user approval.
- Proposals should be minimal and targeted — do not rewrite entire sections.
- If no issues were found, still report the clean sprint but set `BlockedOnUser: false`.
- Include improvements to the **triage decision tree** (Step 7.5) if new failure types were discovered.
