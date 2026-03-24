---
description: End-to-end feature planning — spec, audit, design, tasks, and staging
---

# Feature Planning Workflow

// turbo-all

Take a raw feature idea from spec to an Agent-Ready GitHub issue — no code touched.

> **Note on Multi-Agent Sprints:** Execution happens through `[/worker-agent]`. This workflow handles the **Spec**, **Audit**, and **Plan** phases of Spec-Driven Development.

## Steps

### 1. Build the Spec
Determine if a `spec.md` already exists for this feature.
- **If no spec exists**, collaborate with the user to build one from `.agent/templates/spec.md`:
  1. Collect the raw feature description, target audience, and known constraints.
  2. Draft the spec covering: **Vision & Goals**, **User Requirements**, **User Journeys** (Happy Path, Error Path, Edge Cases), and **KPIs / Definition of Done**.
  3. Present the draft to the user for review. Iterate until approved.
- **If a spec exists**, proceed to Step 2.

### 2. Audit the Spec
Aggressively audit the spec against these critical lenses:
- **Contradictions:** Do any requirements conflict with each other or the goals?
- **Missing Hard Rails:** Are there missing security constraints, performance boundaries, data constraints, or deployment requirements?
- **Incomplete User Journeys:** Are failure states, empty states, loading states, and offline states defined?
- **Ambiguous Success Criteria:** Are the KPIs objectively measurable? Is the Definition of Done crystal clear?

Present a concise list of findings and questions. Integrate the user's answers back into the spec. Once audit is fully resolved, proceed.

### 3. Generate Implementation Plan
Once the spec is bulletproof, generate the technical design and task lists:
- Create a `design.md` draft based on `.agent/templates/design.md`. Document the architecture, components, API contracts, database schemas, and workflows.
- Create a `task.md` draft based on `.agent/templates/task.md`. Break down the execution into small, atomic chunks.
- Present both to the user for review. Update as necessary until approved.

### 4. Stage Issue (Agent-Ready)
Once the user approves the technical design and tasks, formally package it:
1. Consolidate `spec.md`, `design.md`, and `task.md` into the GitHub Issue body.
2. Add relevant metadata tags to the GitHub Issue:
   - The appropriate `domain:` label for isolated worker assignments.
   - Priority labels (e.g., `P0-high`, `P1-medium`).
   - Feature typing or component tags.
3. Mark the issue as **"Agent-Ready"**.

The feature is fully planned! The Lead Orchestrator will assign this issue to a worker agent via `[/orchestrate-sprint]`.

### 5. Retrospective (Self-Improvement)

After the feature issue is staged, analyze this workflow run and propose improvements. This step is mandatory.

#### a) Analyze the Run

| Category | What to Look For |
|---|---|
| **Spec audit quality** | Did the audit catch real gaps, or was it mostly noise? Were critical issues missed? |
| **Template usefulness** | Did `.agent/templates/` have what was needed, or were templates outdated/missing? |
| **User iteration count** | How many review cycles were needed? Could the workflow have asked better upfront questions? |
| **Command failures** | Did any CLI commands (GitHub issue creation, label management) fail or need adaptation? |
| **Ambiguous steps** | Were any steps unclear about expected depth (e.g., how detailed should `design.md` be)? |
| **Redundant steps** | Were any steps unnecessary or could be combined for efficiency? |

#### b) Generate Retrospective Report

```markdown
## 🔄 Workflow Retrospective — Feature Planning

### Run Summary
- **Feature:** #<issue> — <title>
- **Spec review cycles:** <count>
- **Design review cycles:** <count>

### Issues Encountered
| # | Category | Severity | Description |
|---|----------|----------|-------------|
| 1 | <category> | 🔴/🟡/🟢 | <what happened> |

### Proposed Workflow Edits
> ⚠️ These edits require user approval before applying.

#### Proposal 1: <title>
**File:** `.agent/workflows/plan-feature.md`
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
