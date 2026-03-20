# Development Workflow - Quick Reference

## Installation (Plugin Marketplace)

```bash
# 1. Add marketplace (one time)
/plugin marketplace add eljun/workflow-skills

# 2. Install plugins
/plugin install workflow@eljun/workflow-skills

# 3. Configure Playwright MCP for /test (required)
# Add to your project's .mcp.json:
# { "mcpServers": { "playwright": { "command": "npx", "args": ["@playwright/mcp@latest"] } } }

# 4. Create docs folders in your project
mkdir -p docs/task docs/testing docs/features docs/guides docs/changelogs docs/learnings

# 5. TASKS.md is auto-created by /task on first use
```

---

## The Pipeline

```
/task → /implement → /simplify → /test → /document → /ship → /release
```

## Commands

| Command | Model | What it does | Output |
|---------|-------|--------------|--------|
| `/task` | opus | Plan new feature/fix (manual mode) | `docs/task/*.md` |
| `/task auto` | opus | Plan + full automation after approval | Auto pipeline |
| `/implement {task}` | sonnet | Code the task | Working feature |
| `/implement auto {task}` | sonnet | Code + auto-chain through pipeline | Auto pipeline |
| `/simplify {task}` | sonnet | Quality gate — standards + deviation check | PASS/FAIL |
| `/test {task}` | haiku | Run web E2E tests (Playwright) | `docs/testing/*.md` |
| `/document {task}` | haiku | Update docs | `docs/features/*.md` |
| `/ship {task}` | haiku | Create PR | Pull Request |
| `/release` | haiku | Auto-version release | Tag + Changelog |

## Auto Mode

```bash
/task auto              # After approval: implement → simplify → test → document → ship
/implement auto {task}  # Skip planning: simplify → test → document → ship (task doc must exist)
```

**Full automation** through the pipeline
**Test failures:** Auto-retries through implement → simplify → test (max 3 cycles)

## Quality Gate (`/simplify`)

Runs between `/implement` and `/test`. Checks:

| Check | Details |
|-------|---------|
| Coding standards | No `any` types, guard clauses, naming, ≤3 nesting levels, no `console.log` |
| Methodology | TDD/CDD/SOLID compliance (if in task doc) |
| Deviations | Minor → document only, Medium → flag + continue, Major → BLOCK |

PASS → writes Implementation Notes → chains to `/test`
FAIL → reports issues, stops until fixed

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
/release auto     # Same as above
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

## Adding New Skills

```bash
# 1. Create skill folder
mkdir -p {skill-name} && touch {skill-name}/SKILL.md

# 2. Update these files to reference new skill:
#    - task/SKILL.md (Related Skills section)
#    - implement/SKILL.md (Related Skills section)
#    - README.md (Specialized Skills table)
#    - QUICK_REFERENCE.md (this file)
#    - TASKS.md (Skills Reference)
```

See README.md "Adding New Specialized Skills" for full template.

## Key Files

| What | Where |
|------|-------|
| Task docs | `docs/task/*.md` |
| Test reports | `docs/testing/*.md` |
| Retrospectives | `docs/learnings/*.md` |
| Feature docs | `docs/features/*.md` |
| Changelog | `docs/changelogs/CHANGELOG.md` |
| Project knowledge base | `LEARNINGS.md` |
| Status tracker | `TASKS.md` |

## Common Patterns

```bash
# New feature (manual - you control each step)
/task
/clear
/implement 1
/simplify 1
/test 1
/document 1
/ship 1

# New feature (auto - hands-off after approval)
/task auto
# Approve the plan → automation handles the rest

# Auto implement (task doc already exists, skip planning)
/implement auto 1
# → implement → simplify → test → document → ship (automatic)

# Quick fix
/task  # (set Type: bugfix, Version Impact: patch)
/implement the-fix
/simplify the-fix
/test the-fix
/ship the-fix

# Release after multiple ships
/release         # Auto-version from task docs
/release auto    # Same as above
```
