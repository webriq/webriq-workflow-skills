# Development Workflow - Quick Reference

## Installation

```bash
# 1. Add marketplace (one time)
/plugin marketplace add eljun/workflow-skills

# 2. Install plugin
/plugin install workflow@eljun/workflow-skills

# 3. Configure Playwright MCP for /test (required for interactive mode)
# Add to your project's .mcp.json:
# { "mcpServers": { "playwright": { "command": "npx", "args": ["@playwright/mcp@latest"] } } }

# 4. Create docs folders in your project
mkdir -p docs/task docs/testing docs/features docs/guides docs/changelogs docs/learnings
```

---

## The Pipeline

```
/task → /implement → /simplify → /test → /document → /ship → /release
```

## Commands

| Command | What it does | Output |
|---------|--------------|--------|
| `/task` | Plan new feature/fix | `docs/task/*.md` |
| `/task auto` | Plan + full automation after approval | PR when done |
| `/implement {task}` | Code the task | Working feature |
| `/implement auto {task}` | Code + orchestrate full pipeline | PR when done |
| `/simplify {task}` | Quality gate — standards + deviation check | PASS/FAIL |
| `/test {task}` | Run web E2E tests (Playwright) | `docs/testing/*.md` |
| `/document {task}` | Update docs | `docs/features/*.md` |
| `/ship {task}` | Create PR | Pull Request |
| `/release` | Auto-version release | Tag + Changelog |

## Model Routing

`/task` (sonnet) determines the implementation model during planning and writes it to the task document.

| Recommended Model | When |
|-------------------|------|
| `haiku` (default) | UI changes, styling, config, bug fixes, CRUD, single-file additions |
| `sonnet` | Architecture, security, DB schema, cross-cutting concerns, complex logic |

Override at runtime: `/implement --model sonnet {task}` or `/implement {task} "use sonnet"`

## Auto Mode

```bash
/task auto              # After approval: orchestrates implement → simplify → test → document → ship
/implement auto {task}  # Task doc must exist: orchestrates simplify → test → document → ship
```

- **Orchestrator** stays alive, spawns each stage as a background worker
- Workers output a 4-line signal back — orchestrator routes based on result
- Test failures: auto-retries implement → simplify → test (max 3 cycles)
- You are notified with PR link when pipeline completes or stops

## Manual Mode (token-efficient)

```bash
/task                   # Create task doc
/clear                  # ← reset context between every step
/implement {task}
/clear
/simplify {task}
/clear
/test {task}
/clear
/document {task}
/clear
/ship {task}
```

Each skill reads only the task document — it doesn't need the previous session's history. `/clear` between steps keeps input tokens low.

## Quality Gate (`/simplify`)

Runs between `/implement` and `/test`. Checks:

| Check | Details |
|-------|---------|
| Coding standards | No `any` types, guard clauses, naming, ≤3 nesting levels, no `console.log` |
| Methodology | TDD/CDD/SOLID compliance (if in task doc) |
| Deviations | Minor → document only, Medium → flag + continue, Major → BLOCK |

PASS → writes Implementation Notes to task doc → orchestrator assigns test worker
FAIL → reports issues, stops pipeline until fixed

## Task Status Flow

```
PLANNED → IN_PROGRESS → TESTING → APPROVED → READY_TO_SHIP → SHIPPED
```

## Task Types (for changelog)

| Type | Use for |
|------|---------|
| `feature` | New functionality |
| `bugfix` | Bug fixes |
| `enhancement` | Improvements |
| `documentation` | Doc updates |
| `chore` | Maintenance |

## Version Bumps

```bash
/release          # Auto-determine from task docs (RECOMMENDED)
/release patch    # Force v1.0.0 → v1.0.1
/release minor    # Force v1.0.0 → v1.1.0
/release major    # Force v1.0.0 → v2.0.0
```

## Version Impact (set in /task)

| Type | Default Impact | Bump |
|------|----------------|------|
| `feature` | `minor` | v1.0 → v1.1 |
| `bugfix` | `patch` | v1.0.0 → v1.0.1 |
| `enhancement` | `patch` | v1.0.0 → v1.0.1 |
| Breaking change | `major` | v1.0 → v2.0 |

## Specialized Skills

| Skill | When |
|-------|------|
| `/react-best-practices` | React/Next.js code |
| `/postgres-best-practices` | Database queries, RLS, schema |

## Key Files

| What | Where |
|------|-------|
| Task docs | `docs/task/*.md` |
| Task templates | `docs/templates/task-document.md` |
| Test reports | `docs/testing/*.md` |
| Feature docs | `docs/features/*.md` |
| Changelog | `docs/changelogs/CHANGELOG.md` |
| Project knowledge base | `LEARNINGS.md` |
| Status tracker | `TASKS.md` |
