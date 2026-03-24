# Workspace Rules

> Customize this file for your project. Define your brand, design system, file organization, domain boundaries, and auto-approved commands.

## Brand & Voice

- Product name: **[Your Product Name]**
- Brand voice: [Describe your brand's tone and style]

## Design System

- Color palette: [Define CSS variables or design tokens]
- Typography: [Define heading, body, and monospace fonts]
- Icon library: [e.g., `lucide-react`, `heroicons`, `material-icons`]

## Styling

- Vanilla CSS with CSS variables defined in `:root`.
- No inline styles except for dynamic values.
- No TailwindCSS unless explicitly requested.
- Never hardcode colors — always use CSS variables.

## Component Patterns

- Comments explain *why*, not *what*. English only.
- JSDoc/docstrings required for exported functions and components.

## File Organization

- Pages → `src/pages/`
- Components → `src/components/`
- Utilities → `src/utils/`
- Global styles → `src/index.css`

## SDD Methodology

- Every feature or fix must produce three artifacts before implementation: `spec.md`, `design.md`, and `task.md`.
- Use the `[/plan-feature]` workflow to generate all three. It handles spec building, auditing, and implementation planning in one pass.
- Templates live in `.agent/templates/`.
- Finalized artifacts are concatenated into the GitHub Issue body and labeled `Agent-Ready`.
- Execution happens via `[/worker-agent]` inside isolated `git worktree` branches.
- Sprint orchestration is managed via `[/orchestrate-sprint]`.

## Git & Branching

- Commit format: `<type>(#<issue>): <short description>` (conventional commits).
- The `(#<issue>)` tag is mandatory for `feat` and `fix` commits.
- Valid types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`.
- Never commit directly to `main`. Use worktrees: `feat/<issue>-<slug>`.
- Rebase before pushing: `git pull --rebase origin main`.
- A feature is not complete until its GitHub issue is closed.

## Testing

- [Define your test framework and commands here]
- Verify changes locally before requesting review.

## Multi-Agent Protocol

- Each issue is tagged with a `domain:` label. Workers stay within their assigned domain:
  - **domain:frontend** → `src/components/`, `src/pages/`, `src/styles/`
  - **domain:backend** → `api/`, `services/`, `models/`
  - **domain:infra** → `.github/workflows/`, `terraform/`, `docker/`
- Shared files use append-only or labeled-block editing to prevent conflicts.

## Auto-Approved Commands

All of the following are safe to auto-run (`SafeToAutoRun: true`):

- **Build/Test:** `npm run build`, `npm run dev`, `npm install`, `npx vitest run`, `pytest`
- **Git:** `git status`, `git diff`, `git log`, `git branch`, `git add`, `git commit`, `git push`, `git pull --rebase`
- **GitHub CLI:** `gh issue list/view/close/edit/create`, `gh pr create/list/view/merge`
- **Utility:** `cat`, `ls`, `find`, `grep`, `rg`

## Orchestrator Protocol

- Post status comments on issues at assignment and completion.
- Workers must not merge their own PRs.
- If an agent is blocked for more than one sprint cycle, reassign or decompose the task.
