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
Workflow Skills v1.5.2
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

### 1. Gather Context — Full Pipeline Audit

The task doc captures what was **planned**. The actual pipeline (implement → simplify → test) often produces more than that. Your first job is to reconcile plan vs reality across every step.

#### Step 1A: Read the primary sources
```
docs/task/{ID}-{task-name}.md        - What was planned + Implementation Notes from /simplify
docs/testing/{ID}-{task-name}.md     - Test results, failures, retry cycles
```

#### Step 1B: Get the actual file diff
Run this to see every file touched across the entire pipeline:
```bash
git diff --name-only main...HEAD
```

Compare the diff output against the task doc's `## File Changes` section. Anything in the diff but NOT in the task doc was added or changed by `/simplify` or a `/test` retry cycle — these are **pipeline gaps** that need to be captured.

#### Step 1C: Classify each gap

| File in diff but not in task doc | Likely source | Action |
|----------------------------------|---------------|--------|
| Same file, significantly refactored | `/simplify` quality fixes | Note pattern changes in CLAUDE.md if a new convention emerged |
| New helper/utility file extracted | `/simplify` single-responsibility fix | Add to CLAUDE.md Important File Locations if reusable |
| New test fixture or seed file | `/test` setup | Note in LEARNINGS.md if it reveals a testing pattern |
| Additional files changed in retry | `/test` → `/implement` re-run | Document deviation; check if CLAUDE.md needs updating |
| Migration file added mid-cycle | Any step | Always add to CLAUDE.md — migrations are critical context |

**Context Efficiency:** Do NOT do broad codebase exploration. Only open source files that appear in the gap list or are explicitly needed to verify accuracy.

#### Step 1D: Check /simplify's Implementation Notes

The `/simplify` agent writes an `## Implementation Notes` block into the task doc. Read it carefully:
- **"What was built"** — may differ from the original plan
- **"Deviations from plan"** — anything medium or major needs to surface in documentation
- **"Standards check"** — issues that were fixed may have introduced new patterns worth recording in CLAUDE.md

#### Step 1E: Check /test's retry count

In `docs/testing/{ID}-{task-name}.md`, look for how many test cycles ran:
- **0 retries** — implementation was clean; LEARNINGS.md entry should note what worked well
- **1+ retries** — root cause of the failure is a lesson; always capture it in LEARNINGS.md
- **Multiple retries on same issue** — this is a pattern problem; add a "Do Not" entry to CLAUDE.md as well

---

### 2. Identify Documentation Needs

| Change Type | Documentation Needed |
|-------------|---------------------|
| New feature | Feature doc + User guide |
| Enhancement | Update existing docs |
| Bug fix | Update troubleshooting sections |
| API change | Update API reference |
| Pipeline gap (simplify/test added files) | CLAUDE.md + LEARNINGS.md |

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

**Skip the update entirely if:** this task was a pure logic fix, copy change, or styling tweak — no new files, no new directories, no migrations, no new routes, no new packages.

**Never skip if any of these happened** (regardless of task size):
- A migration was added or run
- A new directory was created
- A file was moved or renamed
- A new edge function or API route was added
- A new environment variable is now required
- A pattern was replaced or deprecated

`/implement` should have already captured these — but `/document` must verify and fill any gaps.

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

This retrospective covers the **entire pipeline** — not just what `/task` planned, but everything that happened across implement → simplify → test. Use the pipeline audit from Step 1 (git diff, simplify notes, test report) as your source of truth.

```markdown
# Retrospective: #{ID} - {Task Title}

> **Date:** {date}
> **Quality Gate:** PASS on attempt {N} | PASS first try
> **Test cycles:** {N} — {0 = passed first time, 1-3 = retried}
> **Pipeline gaps:** {None | N files changed by /simplify or /test not in original plan}

## Plan vs Reality

### What was planned
{2-3 sentences from task doc Overview — what was the intent}

### What was actually committed
{From `git diff --name-only main...HEAD` — full list of changed files, not just what the task doc listed}

### Deviations from plan
{None | Minor: {description} | Medium: {description}}

### Files changed outside the plan
{Files that appeared in git diff but were NOT in the task doc's ## File Changes — added by /simplify or /test retry cycles}
{None | List each file and which pipeline step introduced it}

## What /simplify Caught
{Issues the quality gate flagged and fixed — if none, write "None — passed on first attempt"}
{Include: renamed variables, extracted helpers, removed anti-patterns, type fixes}

These changes are direct lessons: future agents doing similar work should apply these standards upfront.

## What /test Revealed
{What failed on first test run and why — if passed first time, write "Tests passed on first run — no retries"}
{If retries happened: root cause of each failure cycle}

Root cause of test failures is the most valuable signal here — it reveals gaps between plan and the real system.

## Key Lessons
1. {Specific, actionable lesson — concrete enough to apply to a future task}
2. {Lesson from /simplify if it caught something worth avoiding next time}
3. {Lesson from /test if failures revealed a non-obvious system behavior}

## Applies To
- Task type: {feature | bugfix | enhancement}
- Tech areas: {e.g., "Supabase auth", "Next.js App Router", "React state management"}
- Pipeline steps with issues: {implement | simplify | test | none}
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

**Append the lesson(s) to the relevant section. Match the source to the right section:**

| Where the lesson came from | Which LEARNINGS.md section |
|----------------------------|---------------------------|
| `/simplify` caught a coding pattern violation | Common Mistakes to Avoid |
| `/simplify` extracted a helper that should be reused | Established Coding Patterns |
| `/test` failed due to a system behavior not in the plan | Tech Stack Notes |
| `/test` retry revealed an integration gap | Common Mistakes to Avoid |
| Files appeared in git diff not in the plan | Tech Stack Notes or Established Coding Patterns |
| Architecture decision made during the pipeline | Architecture & Decisions |

```markdown
## Common Mistakes to Avoid

- **{Mistake in 5 words}**: {What to do instead — 1 sentence}
  *Source: Task #{ID} — {task name} ({date}) — caught by {/simplify | /test}*
```

**Rules for LEARNINGS.md entries:**
- One entry = one actionable lesson, not a summary of the task
- Always credit the pipeline step that caught it (`caught by /simplify`, `revealed by /test retry`)
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
- [ ] Read existing CLAUDE.md — verify `/implement` already captured structural changes
- [ ] Fill any gaps `/implement` missed (migrations, new routes, renamed files, new env vars)
- [ ] Pruned stale entries made obsolete by this task
- [ ] "Do Not" added only for real mistakes caught by /simplify or tests
- [ ] Never skip if migrations, new directories, new routes, or new packages were involved

### Pipeline Audit (do this first)
- [ ] `git diff --name-only main...HEAD` run — full file list captured
- [ ] Cross-referenced diff against task doc's `## File Changes`
- [ ] Pipeline gaps identified (files in diff but not in plan)
- [ ] `/simplify` Implementation Notes read — deviations and fixes noted
- [ ] `/test` retry count checked — root cause of any failures identified

### Retrospective & LEARNINGS.md
- [ ] `docs/learnings/{ID}-{task-name}.md` created
- [ ] Pipeline gaps section filled (files outside original plan)
- [ ] `/simplify` findings documented (patterns caught, fixes applied)
- [ ] `/test` cycle count and root cause documented
- [ ] CLAUDE.md updated for any gaps `/implement` missed (simplify/test introduced changes)
- [ ] Key lesson(s) appended to `LEARNINGS.md` — credited to the pipeline step that caught them
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
