---
name: task
description: Create task documents for new features, updates, or fixes. Creates detailed specs in docs/task/*.md with full implementation context. Updates TASKS.md for tracking. Use when starting any new work.
---

# /task - Task Planning Agent

## Workflow

```
User Request
     ↓
0. Read CLAUDE.md + LEARNINGS.md — get full context before anything else
1. Discuss & clarify requirements
1.5. Determine recommended implementation model (haiku vs sonnet)
2. Research codebase for context (targeted, embed findings)
3. Generate Task ID (next available number)
4. Create docs/task/{ID}-{task-name}.md
5. Add to TASKS.md under "## Planned"
     ↓
Ready for /implement {ID}
```

---

## Step 0: Read Project Knowledge (Before Anything Else)

Read both files before talking to the user or touching the codebase:

- **CLAUDE.md** — stack, conventions, key file locations, "Do Not" rules, architecture decisions already made
- **LEARNINGS.md** — common mistakes to avoid, established patterns to follow, past decisions to honor, test cycle counts
- Together these replace most codebase exploration for planning — read before asking clarifying questions and before any Glob/Grep calls

---

## Step 1: Requirements Gathering

- Ask: What is the expected behavior? Any UI/UX preferences? Priority level?
- Version Impact: `feature` → `minor`, `bugfix/enhancement/chore/documentation` → `patch`. Override to `major` for breaking API changes, schema migrations requiring migration, removing deprecated features, or any change requiring user action.

---

## Step 1.5: Determine Recommended Implementation Model

**Default: haiku** — use for the majority of tasks.

**Escalate to sonnet when:**

| Signal | Examples |
|--------|----------|
| New architecture or system design | New auth system, new data layer, introducing a new pattern |
| Cross-cutting concerns | Changes affecting 5+ unrelated files, or spanning multiple layers (DB + API + UI) |
| Security-sensitive logic | Auth, permissions, RLS policies, token handling |
| Complex business logic | Multi-step workflows, state machines, non-obvious edge cases |
| Database schema changes | Migrations, new tables, RLS, schema refactors |
| Ambiguous or contradictory requirements | Conflicting constraints that need judgment calls |
| High-risk refactor | Replacing existing patterns, removing shared utilities |

**Stick with haiku for:** UI changes, styling, config files, documentation, single-file additions following established patterns, targeted bug fixes, CRUD mirroring existing patterns.

**Record in task document header:**
```
> **Recommended Model:** haiku   ← most tasks
> **Recommended Model:** sonnet  ← complex, architectural, or risky tasks
```

Include a one-line rationale in `## Notes for Implementation Agent` when recommending sonnet.

---

## Step 2: Codebase Research (Targeted — Embed, Don't Just Reference)

- Use specific patterns: `Glob("src/components/auth/**")`, `Grep("useAuthStore", "*.ts")` — not broad sweeps
- After finding relevant files, paste key sections into `## Code Context` in the task doc — do NOT just reference file names

**What to embed and where:**

| Finding | Task doc section |
|---------|-----------------|
| File to create/modify | `## File Changes` table |
| Current code being changed | `## Code Context` — paste it |
| Import patterns needed | `## Code Context` — paste them |
| Similar pattern to follow | `## Code Context` — paste it |
| Non-obvious architectural decision | `## Notes for Implementation Agent` |

If `/implement` would need to read a file to understand it, paste the relevant part into `## Code Context`. The task doc is the single source of truth.

---

## Step 3: Generate Task ID

- Read TASKS.md, find the highest ID across all sections
- Assign next ID = highest + 1; if no tasks exist, start with ID = 1
- Use 3-digit zero-padded format: 001, 002, ... 999

---

## Step 4: Create Task Document

Create `docs/task/{ID}-{task-name}.md` using: `docs/templates/task-document.md`

**Naming:** `001-user-dashboard-redesign.md`, `002-lead-auto-tagging.md`

---

## Step 5: Update TASKS.md

Add to `## Planned`. If TASKS.md missing, create from: `docs/templates/tasks-tracking.md`

---

## Output Checklist

- [ ] Task document created in `docs/task/`
- [ ] All required sections filled
- [ ] Implementation steps are clear and actionable
- [ ] File paths verified to exist (for modifications)
- [ ] Added to TASKS.md "## Planned" section
- [ ] User understands and approves the task

---

## Handoff to /implement

### Manual Mode

```
Task created: #{ID} - {Task Title}
Document: docs/task/{ID}-{task-name}.md
Added to TASKS.md under "Planned"

Next:
  /implement {ID}
  (Optional: /clear first to start fresh session)
```

### Auto Mode

When `/task auto` was invoked and user approves:

1. Set `Automation: auto` in the task document
2. Read `Recommended Model` from the task document header
3. Spawn `/implement auto {ID}` with that model:
   ```
   Task({ subagent_type: "general-purpose", model: "{recommended_model}", prompt: "/implement auto {ID}", run_in_background: true })
   ```
4. Wait for one final signal: pipeline complete or pipeline stopped

```
Task approved. Pipeline starting...

Task: #{ID} - {Task Title}
Orchestrator: /implement ({model} per recommendation)
Pipeline: implement → simplify → test → document → ship

Waiting for pipeline to complete...
```

When the final signal arrives, relay to user:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PIPELINE COMPLETE: #{ID} - {Task Title}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PR: {url}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
Or if stopped:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PIPELINE STOPPED: #{ID} - {Task Title}
Reason: {summary}
Details: {reference path}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
