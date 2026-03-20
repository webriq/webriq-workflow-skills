---
name: document
description: Update project documentation after feature approval. Creates/updates feature docs and user guides. Use after /test passes and user approves. Supports task IDs for easier invocation.
model: haiku
---

# /document - Documentation Agent

> **Model:** haiku (templated documentation work)

## Command Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--help` | `-h` | Show available commands and options |
| `--version` | `-v` | Show workflow skills version |

### Flag Handling

**On `-h` or `--help`:**
```
/document - Documentation Agent

Usage:
  /document {ID}                     Update docs for a task
  /document -h, --help               Show this help message
  /document -v, --version            Show version

Arguments:
  {ID}    Task ID (number) or task filename (e.g., 001-auth-jwt)

Creates/Updates:
  - docs/features/{feature}.md       Technical documentation
  - docs/guides/{feature}.md         User guides (if user-facing)
  - CLAUDE.md                        Project patterns (if needed)

Examples:
  /document 1                        # Document task #1
  /document 001-auth-jwt             # Using task filename

Next: /ship {ID}
```

**On `-v` or `--version`:**
Display:
```
Workflow Skills v1.5.0
https://github.com/eljun/workflow-skills
```

---

## When to Use

Invoke `/document {ID}` when:
- Task has passed testing (`/test` returned PASS)
- User has approved the implementation
- Ready to update project documentation

**Example:** `/document 1` or `/document 001-dashboard-redesign`

## Task ID Resolution

The `{ID}` can be:
- **Numeric ID:** `1`, `2`, `3` → Looks up in TASKS.md, finds matching task document
- **Padded ID:** `001`, `002` → Same as numeric
- **Full filename:** `001-dashboard-redesign` → Direct file reference

## Workflow

```
/document {ID}
       ↓
1. Resolve task ID → find task document
2. Read task document for context
3. Check Automation field (manual | auto)
4. Read test report for verification
5. Update/create feature documentation
6. Update/create user guide (if user-facing)
7. Update CLAUDE.md files if needed
8. Write retrospective → update LEARNINGS.md
9. Move task to "Approved" in TASKS.md
       ↓
┌─── Automation Mode? ───┐
│                        │
▼ Manual                 ▼ Auto
Notify user              Invoke /ship {ID}
Ready for /ship
```

## Auto Mode Behavior

When task document has `Automation: auto`:

After documentation is complete, use Task tool to spawn ship agent with **model: haiku**:
```
Documentation complete: #{ID} - {Task Title}

Updated files:
- docs/features/{feature}.md
- docs/guides/{feature}.md

[AUTO] Spawning /ship with haiku model...
```
`Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/ship {ID}" })`

## Pre-Documentation Checklist

### 1. Gather Context (Primary Sources Only)

Read these files — they contain all the context you need:
```
docs/task/{ID}-{task-name}.md        - Implementation details
docs/testing/{ID}-{task-name}.md     - Test results & verification
```

**IMPORTANT — Context Efficiency:**
These two documents were created by the `/task`, `/implement`, and `/test` agents which already analyzed the codebase. Do NOT perform broad codebase exploration. Only read specific source files if you need to verify implementation details for documentation accuracy.

### 2. Identify Documentation Needs

| Change Type | Documentation Needed |
|-------------|---------------------|
| New feature | Feature doc + User guide |
| Enhancement | Update existing docs |
| Bug fix | Update troubleshooting sections |
| API change | Update API reference |

### 3. Use Templates

Follow the templates defined in this document:
- **Feature Documentation Structure** - Technical feature docs (see below)
- **User Guide Structure** - User-facing guides (see below)

---

## Documentation Types

### 1. Feature Documentation

**Location:** `docs/features/{FEATURE}.md`
**Audience:** Developers
**Purpose:** Technical implementation details

**When to create/update:**
- New feature → Create new doc
- Enhancement → Update existing doc
- Significant change → Update relevant sections

### 2. User Guide

**Location:** `docs/guides/{feature}.md`
**Audience:** End users (business owners, customers)
**Purpose:** How to use the feature

**When to create/update:**
- User-facing feature → Create/update guide
- UI changes → Update screenshots/steps
- New functionality → Add new sections

### 3. CLAUDE.md — Living Project Brain

**Location:** `/CLAUDE.md` — auto-loaded by Claude Code on every session, zero cost

CLAUDE.md is the most token-efficient document in the entire project. Every agent reads it for free before any tool call happens. That means stale entries in CLAUDE.md actively mislead agents — they'll follow outdated patterns with full confidence.

**The discipline: read → prune → update (in that order)**

Before adding anything, read the current CLAUDE.md and ask for each existing entry:
- Is this still true based on what was just built?
- Did this task rename a file, change a folder structure, migrate a schema, or replace a pattern?
- If yes → update or remove the stale entry before adding new ones

**Never just append. CLAUDE.md that only grows becomes a liability.**

---

**The bar for a CLAUDE.md entry:**

An entry earns its place if a future agent would either:
- Make a mistake without it (wrong pattern, wrong file, wrong approach)
- Waste time discovering it (file location, non-obvious convention)
- Make a decision already made (architecture choice, tool selection)

If the answer is "probably fine without it" — don't add it.

---

**Maintain this structure:**

```markdown
# {Project Name}

## Stack
{Only what's non-obvious or frequently relevant to agents}
- Framework: Next.js 14 App Router (NOT Pages Router)
- Database: Supabase (PostgreSQL) — client at src/lib/supabase/client.ts
- Auth: Supabase Auth — helpers at src/lib/auth.ts
- State: Zustand — stores at src/store/
- Styling: Tailwind CSS

## Project Structure
{Update this when directories are created, renamed, or removed}
src/
  app/           # App Router pages and layouts
  components/
    ui/          # Primitive components (Button, Input, Modal)
    features/    # Feature-specific components
  lib/           # Integrations (supabase/, stripe/, etc.)
  hooks/         # Custom React hooks
  store/         # Zustand stores

## Key Conventions
{Only conventions that are actively enforced or non-obvious}
- Components: PascalCase files and exports
- Hooks: use* prefix, always in src/hooks/
- DB queries: always go through src/lib/supabase/ helpers, never raw client calls in components
- {Add when /simplify catches a violation for the second time — that means it's a real pattern}

## Important File Locations
{Files that agents always need to find — update when files move}
- Supabase client: src/lib/supabase/client.ts
- Auth helpers: src/lib/auth.ts
- Route middleware: src/middleware.ts
- Environment types: src/types/env.d.ts

## Do Not
{Hard rules — each one from a real mistake, not speculation}
- Do not import Supabase client directly in components — use lib/supabase/ helpers
- Do not use `any` types — caught by /simplify, will fail quality gate
- {Add only when something was caught by the quality gate or broke in tests}

## Architecture Decisions
{Decisions made so future agents don't re-open the debate}
- Using Zustand over Redux: simpler API for this app's state complexity
- App Router over Pages Router: required for server components
- {Date of decision so agents can judge if it's still current}
```

---

**Triggers for a CLAUDE.md update after this task:**

| What happened | What to update |
|---------------|----------------|
| New directory created | Project Structure |
| File moved or renamed | Important File Locations |
| Database migration ran | Stack section, any schema notes |
| New package installed | Stack section |
| /simplify caught a repeated violation | Do Not |
| Architecture debate resolved | Architecture Decisions |
| Convention violated and then fixed | Key Conventions |
| Old pattern replaced by new one | Remove old entry, add new |

**Skip the update entirely if:** this task was a small bugfix or UI change that didn't touch architecture, conventions, or file structure.

### 4. Retrospective & LEARNINGS.md

**Locations:**
- `docs/learnings/{ID}-{task-name}.md` — Full retrospective for this task
- `LEARNINGS.md` — Project-wide knowledge base (read by `/task` and `/implement` on every run)

**Always write after documentation is complete.** This is what makes the workflow self-learning — each completed task teaches future agents what to do and what to avoid.

---

## Feature Documentation Structure

```markdown
# Feature: {Feature Name}

> **Status:** PRODUCTION
> **Last Updated:** {Date}

## Overview

{Brief 2-3 sentence description}

---

## User Journey

### For Customers
1. Step 1
2. Step 2

### For Business Owners
1. Step 1
2. Step 2

---

## Architecture

### File Structure
{Accurate file tree - VERIFY paths exist}

### Database Schema
{SQL with comments, if applicable}

### API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|

---

## Implementation Details

### Key Components
| Component | Location | Purpose |
|-----------|----------|---------|

### Technical Notes
- Important detail 1
- Important detail 2

---

## Related Features
- [Link to related feature]
```

**Target length:** 200-400 lines

---

## User Guide Structure

```markdown
# {Feature Name} Guide

> {Brief intro sentence}

## Quick Start
{1-2 paragraph overview}

---

## For Customers

### How to {Primary Action}
1. Step with context
2. Step with context

### Tips
- Tip 1
- Tip 2

---

## For Business Owners

### How to {Admin Action}
1. Step with context

### Settings
| Setting | Location | What it does |
|---------|----------|--------------|

---

## FAQ

**Q: Question?**
A: Answer.

---

## Troubleshooting

**Issue:** Description
**Solution:** How to fix

---

## Related Guides
- [Link]
```

**Target length:** 100-200 lines

---

## Retrospective Writing

This is the self-learning step. Every completed task generates signal — plan vs reality, quality gate findings, test failures, decisions made. Writing it down means future agents don't repeat the same mistakes.

### Step 1: Write Task Retrospective

**File:** `docs/learnings/{ID}-{task-name}.md`

Gather from sources already read (task doc, simplify Implementation Notes, test report):

```markdown
# Retrospective: #{ID} - {Task Title}

> **Date:** {date}
> **Quality Gate:** PASS on attempt {N} | PASS first try
> **Test cycles:** {N} — {0 = passed first time, 1-3 = retried}

## Plan vs Reality

### What was planned
{2-3 sentences from task doc Overview — what was the intent}

### What was delivered
{What actually changed — from git diff or Implementation Notes}

### Deviations from plan
{None | Minor: {description} | Medium: {description}}

## What the Quality Gate Caught
{Issues /simplify flagged — if none, write "None — implementation met standards on first pass"}

These are direct lessons: if the quality gate caught something, future agents doing similar work should know to avoid it upfront.

## What Tests Revealed
{What failed on first test run and why — if tests passed first time, write "Tests passed on first run"}

Root cause of any test failures is the most valuable lesson here.

## Key Lessons
1. {Specific, actionable lesson — concrete enough to apply to a future task}
2. {Another lesson if applicable}

## Applies To
- Task type: {feature | bugfix | enhancement}
- Tech areas: {e.g., "Supabase auth", "Next.js App Router", "React state management"}
```

### Step 2: Update LEARNINGS.md

**File:** `LEARNINGS.md` (project root)

Extract the most reusable insight from the retrospective and append it. This file is read by `/task` and `/implement` at the START of every run — keep each entry concise and actionable.

**If LEARNINGS.md doesn't exist, create it:**

```markdown
# Project Learnings

> Auto-updated by /document after each completed task.
> Read by /task and /implement before starting any work.

---

## Architecture & Decisions

| Decision | Rationale | Date | Task |
|----------|-----------|------|------|

---

## Established Coding Patterns

{Patterns that have proven to work in this codebase}

---

## Common Mistakes to Avoid

{Things the quality gate or tests caught — don't repeat these}

---

## Testing Patterns

{What works and what doesn't when testing this project}

---

## Tech Stack Notes

{Non-obvious things about the stack that future agents should know}
```

**Append the lesson(s) to the relevant section:**

```markdown
## Common Mistakes to Avoid

- **{Mistake in 5 words}**: {What to do instead — 1 sentence}
  *Source: Task #{ID} — {task name} ({date})*
```

**Rules for LEARNINGS.md entries:**
- One entry = one actionable lesson, not a summary of the task
- If the lesson is "nothing went wrong," write a pattern that worked well instead
- Never duplicate — check if a similar lesson already exists before appending
- Keep entries scannable: bold the key phrase, explain in one sentence

---

## Documentation Checklist

### Feature Doc
- [ ] Follows template structure
- [ ] File paths verified to exist
- [ ] API endpoints are accurate
- [ ] Schema matches database
- [ ] Status set to PRODUCTION
- [ ] Last Updated date set
- [ ] Under 400 lines

### User Guide
- [ ] Follows template structure
- [ ] Clear step-by-step instructions
- [ ] FAQ answers common questions
- [ ] Troubleshooting section included
- [ ] Under 200 lines

### CLAUDE.md Updates
- [ ] Read existing CLAUDE.md before touching it
- [ ] Pruned any entries made stale by this task (moved files, changed patterns, replaced approaches)
- [ ] Stack updated if new dependency added
- [ ] Project Structure updated if directories created or renamed
- [ ] Important File Locations updated if key files moved
- [ ] Key Conventions updated only if a pattern is now actively enforced
- [ ] "Do Not" added only for real mistakes caught by /simplify or tests
- [ ] Skipped entirely if task was a small change with no architectural impact

### Retrospective & LEARNINGS.md
- [ ] `docs/learnings/{ID}-{task-name}.md` created
- [ ] Quality gate findings documented
- [ ] Test cycle count and root cause documented
- [ ] Key lesson(s) appended to `LEARNINGS.md`
- [ ] No duplicate entries in LEARNINGS.md

---

## Update TASKS.md

Move task to "Approved" section:

```markdown
## Approved

| ID | Task | Task Doc | Feature Doc | Test Report | Approved |
|----|------|----------|-------------|-------------|----------|
| 1 | Quick Actions Redesign | [001-quick-actions.md](docs/task/001-quick-actions.md) | [feature](docs/features/...) | [001-quick-actions.md](docs/testing/001-quick-actions.md) | Jan 25 |
```

---

## What to Include

- **Current state only** - Document what exists now
- **Accurate file paths** - Verify paths exist
- **Working examples** - Only code that matches production
- **Clear user journeys** - What users actually do

## What to Exclude

| Remove | Why |
|--------|-----|
| Development logs | Move to changelogs |
| Before/after comparisons | Only document current state |
| "Lessons learned" | Dev notes, not docs |
| Speculative future phases | Keep minimal |
| Full component code | Code lives in codebase |

---

## Handoff to /ship

**Check the task document for `Automation: auto` field.**

### Manual Mode
```
Documentation updated for: #{ID} - {Task Title}

Updated files:
- docs/features/{feature}.md (created/updated)
- docs/guides/{feature}.md (created/updated)
- CLAUDE.md (if updated)
- docs/learnings/{ID}-{task-name}.md (retrospective)
- LEARNINGS.md (lesson appended)

Task moved to "Approved" in TASKS.md

Next Steps:
  /ship {ID}              # e.g., /ship 1
  /ship {ID}-{task-name}  # e.g., /ship 001-auth-jwt
```

### Auto Mode
```
Documentation updated for: #{ID} - {Task Title}

Updated files:
- docs/features/{feature}.md (created/updated)
- docs/guides/{feature}.md (created/updated)
- CLAUDE.md (if updated)
- docs/learnings/{ID}-{task-name}.md (retrospective)
- LEARNINGS.md (lesson appended)

Task moved to "Approved" in TASKS.md

[AUTO] Spawning /ship with haiku model...
```
Use Task tool: `Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/ship {ID}" })`

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/implement` | If issues found during doc review |
| `/ship` | After documentation complete |

## Recommended Plugins (Install Separately)

These plugins must be installed separately. **Once installed, they MUST be invoked** — do not skip them:

| Plugin | Install From | When to Invoke |
|--------|--------------|----------------|
| `vercel-react-best-practices` | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | Reference React patterns for docs |
| `supabase-postgres-best-practices` | [supabase/agent-skills](https://github.com/supabase/agent-skills) | Reference database patterns for docs |
