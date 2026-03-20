# Claude Code Development Workflow Skills

> A structured skill-based development pipeline for consistent, high-quality feature delivery.

**Version:** 1.5.0 | [Changelog](#)

## Installation

### Step 1: Install Workflow Plugin

#### Option A: Via Plugin Marketplace (Recommended)

Add the marketplace to Claude Code and install the workflow plugin:

```bash
# In Claude Code, run:
/plugin

# Then select:
# 1. "Marketplaces" tab
# 2. "Add Marketplace"
# 3. Enter: eljun/claude-skills
# 4. Go to "Plugins" tab
# 5. Enable "workflow"
```

#### Option B: Via npx skills CLI (Alternative)

If you're having issues with the plugin marketplace, use the CLI:

```bash
# Install globally
npx skills add eljun/claude-skills -y -g

# Or install to project only
npx skills add eljun/claude-skills -y
```

**Flags:**
| Flag | Description |
|------|-------------|
| `-y` | Skip confirmation prompts |
| `-g` | Install globally (available in all projects) |

This installs the core workflow skills:

| Skill | Purpose |
|-------|---------|
| `/task` | Plan features and create task documents |
| `/implement` | Implement tasks step by step |
| `/simplify` | Quality gate — coding standards + deviation check |
| `/test` | Run E2E tests via Playwright |
| `/document` | Generate feature docs and guides |
| `/ship` | Create PRs and prepare deployment |
| `/release` | Create versioned releases with changelogs |

### Step 2: Install Companion Skills (Recommended)

The workflow skills reference these specialized skills for React/Next.js and Supabase/PostgreSQL projects. Installation is **optional but recommended**. Once installed, the workflow skills will **always invoke them** — they are not skippable.

Run these commands in your **project directory**:

```bash
# React/Next.js best practices (from Vercel)
npx skills add vercel-labs/agent-skills

# Supabase/PostgreSQL best practices (from Supabase)
npx skills add supabase/agent-skills
```

For each command, follow the prompts:

1. **Install to** → Select specific agents
2. **Select skills** → Choose the skills you want (e.g., `vercel-react-best-practices`, `supabase-postgres-best-practices`)
3. **Select agents to install to** → Claude Code
4. **Installation scope** → Project
5. **Installation method** → Symlink (Recommended)

| Skill | Source | Purpose |
|-------|--------|---------|
| `/vercel-react-best-practices` | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | React/Next.js performance optimization |
| `/supabase-postgres-best-practices` | [supabase/agent-skills](https://github.com/supabase/agent-skills) | Database queries, RLS, schema design |

When installed, the workflow skills (`/task`, `/implement`, etc.) will automatically reference these during relevant tasks.

### Step 3: Configure Playwright for `/test`

The `/test` skill supports two modes with different requirements:

#### Option A: Interactive Mode (Default)

For visual browser testing where you see each step in real-time.

**Requires Playwright MCP.** Add this to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

**Usage:** `/test {task-name}`

#### Option B: CI Mode (Headless)

For headless testing with generated test scripts. Faster, supports parallel execution.

**Requires Playwright package** (no MCP needed):

```bash
# Install Playwright
npm install -D @playwright/test

# Install browsers (first time only)
npx playwright install
```

**Usage:**
- `/test --ci {task-name}` - keeps test scripts (default)
- `/test --ci --cleanup {task-name}` - deletes test scripts after

#### Comparison

| Feature | Interactive | CI Mode |
|---------|-------------|---------|
| Setup | Playwright MCP | `@playwright/test` package |
| Visibility | See browser in real-time | Headless (background) |
| Speed | Sequential | Parallel execution |
| Test scripts | Not created | Kept by default for regression |
| Best for | Debugging, demos | Large test suites, automation |

#### CI Mode Flags

| Command | Behavior |
|---------|----------|
| `/test --ci {task}` | Run tests, **keep scripts** for regression (default) |
| `/test --ci --cleanup {task}` | Run tests, **delete scripts** after (one-time verification) |

> **Tip:** Use `--ci` (default) for core features to build regression test suite. Use `--ci --cleanup` for minor changes that don't need long-term testing.

### Step 4: Project Setup

Create the required folders in your project:

```bash
mkdir -p docs/task docs/testing docs/features docs/guides docs/changelogs docs/learnings tests/e2e
```

| Folder | Purpose |
|--------|---------|
| `docs/task/` | Task documents created by `/task` |
| `docs/testing/` | Test reports created by `/test` |
| `docs/features/` | Feature documentation created by `/document` |
| `docs/guides/` | User guides created by `/document` |
| `docs/changelogs/` | Changelog created by `/release` |
| `docs/learnings/` | Per-task retrospectives created by `/document` |
| `tests/e2e/` | E2E test scripts created by `/test --ci` |
| `TASKS.md` | Task tracker (created automatically by `/task`) |
| `LEARNINGS.md` | Project knowledge base — read by `/task` and `/implement` on every run |

## Quick Start

```bash
# Get help for any skill
/task --help              # or -h
/implement --help
/test -v                  # Show version

# Manual Mode - You control each step
/task                     # Creates task #1 → docs/task/001-feature-name.md
/implement 1              # Implement task #1 (or use full name: 001-feature-name)
/simplify 1               # Quality gate — check standards before testing
/test 1                   # Test task #1
/document 1               # Document task #1
/ship 1                   # Ship task #1

# Multi-task Mode - Parallel implementation
/implement -m 1 2 3       # Spawn parallel agents for tasks 1, 2, and 3

# Auto Mode - Full automation after plan approval
/task auto
# → After you approve, runs: implement → simplify → test → document → ship automatically

# Auto Mode - Skip planning, auto-chain from implement
/implement auto 1
# → Task doc must exist. Runs: implement → simplify → test → document → ship automatically
```

### Task IDs

Tasks are assigned simple numeric IDs (1, 2, 3...) for easy reference:

```bash
# These are equivalent:
/implement 1
/implement 001-auth-jwt
```

Task files are named with zero-padded IDs for sorting: `001-auth-jwt.md`, `002-fix-login.md`

---

## Workflow Overview

### Manual Mode (`/task`)

```
┌────────┐   ┌────────────┐   ┌──────────┐   ┌────────┐   ┌──────────┐   ┌────────┐   ┌──────────┐
│ /task  │ → │ /implement │ → │/simplify │ → │ /test  │ → │/document │ → │ /ship  │ → │ /release │
└────────┘   └────────────┘   └──────────┘   └────────┘   └──────────┘   └────────┘   └──────────┘
    │              │                │              │              │              │              │
    ↓              ↓                ↓              ↓              ↓              ↓              ↓
docs/task/    In Progress      PASS/FAIL       Testing       Approved      Ready to      Shipped
  *.md                                                                       Ship        + Changelog
```

**Manual Checkpoints:**
- `/simplify` FAIL → **Fix code**, re-run `/simplify` (blocks before testing)
- After `/test` → **You approve** before `/document`
- After `/document` → **You decide** when to `/ship`
- After `/ship` → **You decide** when to `/release`

### Auto Mode (`/task auto` or `/implement auto`)

```
/task auto → User approves plan       /implement auto {task}
    ↓                                        ↓
/implement (sonnet)                    Implement code
    ↓                                        ↓
/simplify (sonnet) ◄─────────────────── /simplify (quality gate)
    │                                        │
PASS → /test                           PASS → /test
FAIL → fix & retry (max 3)             FAIL → fix & retry (max 3)
    │
/test (haiku) ←── Playwright E2E
    │
PASS → /document
FAIL → /implement (with test report) → /simplify → /test
    │
    ▼
/ship → PR created
```

**Auto Mode Features:**
- **Two entry points:** Start from `/task auto` (full pipeline) or `/implement auto` (skip planning)
- **Quality gate:** `/simplify` validates coding standards and deviations before wasting a test run
- **Full automation:** Runs through the remaining pipeline automatically
- **Test failures:** Auto-retries through implement → simplify → test (max 3 cycles)
- **PR creation:** Creates PR and notifies you - you decide when to merge

### Model Configuration

Each skill uses an optimized model for its task complexity:

| Skill | Model | Reason |
|-------|-------|--------|
| `/task` | **opus** | Complex planning requires advanced reasoning |
| `/implement` | **sonnet** | Strong coding with better cost efficiency than opus |
| `/simplify` | **sonnet** | Reasoning needed to evaluate code quality and detect deviations |
| `/test` | **haiku** | Straightforward test execution from BDD acceptance criteria |
| `/document` | **haiku** | Templated documentation work |
| `/ship` | **haiku** | Scripted deployment commands |
| `/release` | **haiku** | Scripted release commands |

---

## Skills Reference

All skills support `--help` (`-h`) and `--version` (`-v`) flags.

### 1. `/task` - Task Planning

**Purpose:** Discuss requirements and create detailed task documents.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |
| `auto` | Enable automated pipeline |

**Output:**
- Task ID (e.g., #1)
- Task document in `docs/task/{ID}-{task-name}.md`
- Entry in TASKS.md "Planned" section

**Example:**
```
User: I want to add dark mode to the app
Claude: /task
→ Discusses requirements, development approach (TDD/CDD/Standard)
→ Writes BDD acceptance criteria (Given/When/Then)
→ Creates docs/task/001-dark-mode.md
→ Adds to TASKS.md as #1
→ Next: /implement 1
```

---

### 2. `/implement` - Implementation

**Purpose:** Implement tasks following the task document.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |
| `-m, --multi` | Multi-task parallel execution |
| `auto` | Auto-chain: simplify → test → document → ship |

**Input:** Task ID (number) or task filename

**Options:**
| Command | Behavior |
|---------|----------|
| `/implement {ID}` | Implement single task |
| `/implement auto {ID}` | Implement, then auto-chain |
| `/implement -m {ID1} {ID2}` | Parallel agents for multiple tasks |

**Example:**
```
/implement 1                    # Implement task #1
/implement 001-dark-mode        # Using filename
/implement auto 1               # With auto-chain
/implement -m 1 2 3             # Parallel implementation
→ Next: /simplify 1
```

---

### 3. `/simplify` - Quality Gate

**Purpose:** Validate coding standards and detect plan deviations before testing.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |

**Input:** Task ID (number) or task filename

**Checks:**
| Check | What it validates |
|-------|-------------------|
| Coding standards | No `any` types, guard clauses, naming, nesting ≤ 3 levels, no `console.log` |
| Methodology | TDD/CDD/SOLID compliance (if specified in task doc) |
| Deviations | Minor → document, Medium → flag, Major → BLOCK |

**Result:**
- **PASS** → writes Implementation Notes to task doc → chains to `/test`
- **FAIL** → reports issues, stops until fixed

**Example:**
```
/simplify 1
→ Reads task doc + git diff
→ Checks standards on changed files
→ Classifies any deviations from plan
→ Writes Implementation Notes
→ PASS: chains to /test 1
→ FAIL: lists issues to fix
```

---

### 4. `/test` - Web E2E Testing

**Purpose:** Test **web** implementations using Playwright, following BDD acceptance criteria.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |
| `--ci` | CI mode: headless with scripts |
| `--cleanup` | Delete scripts after (with --ci) |

**Input:** Task ID (number) or task filename

**Modes:**
| Mode | Command | Test Scripts |
|------|---------|--------------|
| Interactive | `/test {ID}` | Not created |
| CI (default) | `/test --ci {ID}` | Kept for regression |
| CI + cleanup | `/test --ci --cleanup {ID}` | Deleted after test |

**Example:**
```
/test 1                         # Interactive visual testing
/test --ci 1                    # CI mode, scripts kept
/test --ci --cleanup 1          # CI mode, scripts deleted
→ Next: /document 1
```

---

### 5. `/document` - Documentation

**Purpose:** Update project documentation after approval.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |

**Input:** Task ID (number) or task filename

**Example:**
```
/document 1
→ Creates/updates feature doc
→ Creates/updates user guide
→ Moves to "Approved" section
→ Next: /ship 1
```

---

### 6. `/ship` - Deployment

**Purpose:** Create PRs and prepare for deployment.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |

**Input:** Task ID (number) or task filename

**Output:**
- Git branch: `feature/{ID}-{task-name}`
- Pull Request
- Task in "Ready to Ship" section

**Example:**
```
/ship 1
→ Runs pre-deployment checks
→ Creates branch: feature/1-dark-mode
→ Creates PR
→ Next: Merge PR, then /release
```

---

### 7. `/release` - Versioned Releases

**Purpose:** Create versioned releases with consolidated changelogs.

**Flags:**
| Flag | Description |
|------|-------------|
| `-h, --help` | Show usage and options |
| `-v, --version` | Show workflow skills version |
| `auto` | Auto-determine version from task types |
| `patch/minor/major` | Force version bump type |

**Example:**
```
/release                        # Auto-detect version
/release minor                  # Force minor bump
/release v1.2.0                 # Explicit version
→ Creates changelog + git tag + GitHub release
```

---

## Task Document Template

When `/task` creates a task, it uses this structure:

**Filename:** `docs/task/{ID}-{task-name}.md` (e.g., `001-dark-mode.md`)

```markdown
# {Task Title}

> **ID:** {number}
> **Status:** PLANNED | TESTING | APPROVED | SHIPPED
> **Priority:** HIGH | MEDIUM | LOW
> **Type:** feature | bugfix | enhancement | documentation | chore
> **Version Impact:** minor | patch | major
> **Created:** {Date}
> **Platform:** Web
> **Automation:** manual | auto

## Overview
{2-3 sentence description}

## Development Approach

**Methodology:** TDD | CDD | Standard
**Rationale:** {1 sentence — why this fits the task type}

## Requirements
### Must Have
- [ ] Requirement 1
- [ ] Requirement 2

### Nice to Have
- [ ] Optional requirement

## Current State
{Description of how things work now}

## Proposed Solution
{Description of the implementation approach}

### File Changes
| Action | File | Description |
|--------|------|-------------|
| CREATE | `path/to/new.tsx` | New component |
| MODIFY | `path/to/existing.tsx` | Add functionality |

## Implementation Steps
### Step 1: {Title}
{Detailed instructions}

### Step 2: {Title}
{Detailed instructions}

## Acceptance Criteria

### Happy path
- [ ] Given {starting state}, when {action}, then {observable outcome}

### Error states
- [ ] Given {condition}, when {action}, then {specific error or behavior}

### Edge cases
- [ ] {Specific scenario that could break things}

### Test setup
- **URL:** {entry point for testing}
- **Test credentials:** {email/password or "N/A"}
- **Setup required:** {seed data, env vars, migrations — or "None"}

## Notes for Implementation Agent
{Important context}
```

---

## TASKS.md Structure

The workflow uses these sections in TASKS.md. **The `/task` skill auto-creates this file if it doesn't exist.**

All tables include an **ID** column for easy task reference.

```markdown
## Planned
Tasks ready for `/implement {ID}`. Created via `/task`.
| ID | Task | Priority | Task Doc | Created |

## In Progress
| ID | Task | Started | Task Doc | Status |

## Testing
Tasks being tested via `/test`. Returns to `/implement` if FAIL.
| ID | Task | Task Doc | Test Report | Status |

## Approved
Tested and approved. Ready for `/document` then `/ship`.
| ID | Task | Task Doc | Feature Doc | Test Report | Approved |

## Ready to Ship
PRs created via `/ship`. **Items stay here until `/release`** (even after merge).
| ID | Task | Branch | PR | Merged | Task Doc |

## Shipped
Released items. Only `/release` moves items here with version.
| ID | Task | PR | Release | Shipped |
```

### Release Tracking Flow

```
/ship creates PR → "Ready to Ship" (Merged = No)
       ↓
PR merged → Update "Merged" column (Merged = ✅ Jan 26)
       ↓
/release → Moves merged items to "Shipped" with version (v1.2.0)
```

This ensures `/release` always knows exactly which features to include and which release version they belong to.

---

## Specialized Skills (External)

These skills provide domain-specific best practices and can be invoked during `/implement`. Install them from their original sources:

| Skill | Purpose | Install From |
|-------|---------|--------------|
| `/vercel-react-best-practices` | React/Next.js performance optimization | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) |
| `/supabase-postgres-best-practices` | Database queries, RLS, schema design | [supabase/agent-skills](https://github.com/supabase/agent-skills) |

---

## Adding New Specialized Skills

### Skill Types

| Type | Purpose | Examples |
|------|---------|----------|
| **Workflow** | Pipeline stages | `/task`, `/implement`, `/simplify`, `/test`, `/document`, `/ship`, `/release` |
| **Specialized** | Domain best practices (external) | `/vercel-react-best-practices`, `/supabase-postgres-best-practices` |

**Specialized skills** are invoked by workflow skills when relevant. To add a new one:

### Step 1: Create the Skill Folder

```bash
mkdir -p {skill-name}
touch {skill-name}/SKILL.md
```

### Step 2: Write the SKILL.md

Use this template for best-practices skills:

```markdown
---
name: {skill-name}
description: {Technology} best practices for AI agents. Use when {trigger conditions}.
---

# /{skill-name} - {Technology} Best Practices

## When to Use

Invoke `/{skill-name}` when:
- Writing or modifying {technology} code
- Reviewing {technology} patterns
- Optimizing {technology} performance

## Priority Categories

| Priority | Category | Impact |
|----------|----------|--------|
| 1 | {Category} | CRITICAL |
| 2 | {Category} | HIGH |
| 3 | {Category} | MEDIUM |

---

## 1. {Category Name} (CRITICAL)

### 1.1 {Rule Name}

**BAD:**
```{language}
// Anti-pattern example
```

**GOOD:**
```{language}
// Best practice example
```

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/task` | Planning {technology} features |
| `/implement` | Implementing {technology} code |
```

### Step 3: Update Workflow Skills

Add the new skill to the "Related Skills" section in these files:

**Files to update:**
- `task/SKILL.md` - Related Skills section
- `implement/SKILL.md` - Related Skills section

### Step 4: Update Documentation

**Update these files:**

1. **README.md** - Add to the Specialized Skills table (this file)
2. **QUICK_REFERENCE.md** - Add to the Specialized Skills section
3. **TASKS.md** - Add to the Skills Reference section

### Example: Adding `/tailwind-best-practices`

```bash
# 1. Create folder
mkdir -p tailwind-best-practices

# 2. Create SKILL.md with best practices content

# 3. Update task/SKILL.md Related Skills:
| `/tailwind-best-practices` | Tailwind CSS patterns |

# 4. Update implement/SKILL.md Related Skills:
| `/tailwind-best-practices` | Tailwind CSS code |

# 5. Update README.md Specialized Skills table:
| `/tailwind-best-practices` | Tailwind CSS utility patterns | Styling with Tailwind |

# 6. Update QUICK_REFERENCE.md:
| `/tailwind-best-practices` | Tailwind CSS styling |

# 7. Update TASKS.md Skills Reference:
| `/tailwind-best-practices` | Tailwind CSS patterns |
```

### Skill Ideas

Common specialized skills you might want to add:

| Skill | Purpose |
|-------|---------|
| `/tailwind-best-practices` | Tailwind CSS utility patterns |
| `/typescript-best-practices` | TypeScript type safety patterns |
| `/testing-best-practices` | Unit/integration test patterns |
| `/api-best-practices` | REST/GraphQL API design |
| `/security-best-practices` | Security patterns and validation |
| `/accessibility-best-practices` | WCAG accessibility patterns |

---

## Best Practices

### For Planning (`/task`)
- Be specific about requirements
- Include BDD acceptance criteria (Given/When/Then)
- Choose the right methodology (TDD for logic, CDD for UI, Standard for APIs)
- Add the `Type` field for changelog categorization

### For Implementation (`/implement`)
- Follow the task document step by step
- Write modular, single-responsibility functions
- No `any` types, no deep nesting, no `console.log` in production paths
- Don't expand scope without approval

### For Quality Gate (`/simplify`)
- Run after every implement before testing
- Major deviations block the pipeline — re-plan with `/task`
- Medium deviations are flagged but don't block

### For Testing (`/test`)
- Tests follow the BDD acceptance criteria in the task doc
- Happy path first, then error states and edge cases
- Document any issues found specifically

### For Documentation (`/document`)
- Follow the templates defined in the `/document` skill
- Verify file paths exist
- Include both developer and user perspectives
- Keep it concise (feature: 200-400 lines, guide: 100-200 lines)

### For Shipping (`/ship`)
- Run all checks before creating PR
- Include task doc and test report links in PR
- Don't force push to main

### For Releases (`/release`)
- Batch related changes together
- Use semantic versioning appropriately
- Write user-focused changelog entries

---

## Troubleshooting

### Plugin marketplace installation fails
If `/plugin` marketplace method doesn't work, use the CLI instead:
```bash
npx skills add eljun/claude-skills -y -g
```

### "Task not found"
- Check TASKS.md for the task entry
- Verify task document exists in `docs/task/`

### "/simplify fails with standards issues"
- Fix the reported issues in the listed files
- Re-run `/implement {ID}` to fix, then `/simplify {ID}` again

### "Tests failing"
- Review the test report in `docs/testing/`
- Return to `/implement` with the test report context
- `/simplify` will verify the fix addresses the failure before re-testing

### "Build fails during /ship"
- Fix the issues locally
- Re-run `/ship` after fixes

### "Wrong section in TASKS.md"
- Manually move the task to the correct section
- Or ask Claude to update the status

---

## File Locations

| Content | Location | Example |
|---------|----------|---------|
| Task documents | `docs/task/{ID}-{name}.md` | `docs/task/001-auth-jwt.md` |
| Test reports | `docs/testing/{ID}-{name}.md` | `docs/testing/001-auth-jwt.md` |
| Task retrospectives | `docs/learnings/{ID}-{name}.md` | `docs/learnings/001-auth-jwt.md` |
| E2E test scripts | `tests/e2e/{ID}-{name}/` | `tests/e2e/001-auth-jwt/` |
| Feature docs | `docs/features/*.md` | `docs/features/authentication.md` |
| User guides | `docs/guides/*.md` | `docs/guides/authentication.md` |
| Changelog | `docs/changelogs/CHANGELOG.md` | |
| Project knowledge base | `LEARNINGS.md` | Read by every agent at start |
| Workflow tracker | `TASKS.md` | |
| Skills version | `plugins/workflow/VERSION` | `1.5.0` |

---

## Questions?

- Check the individual skill files in `.claude/skills/*/SKILL.md`
- Review `CLAUDE.md` for project-specific patterns
- Check `TASKS.md` for current task status
