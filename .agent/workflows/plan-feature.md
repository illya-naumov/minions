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
