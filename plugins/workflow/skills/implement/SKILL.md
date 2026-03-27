---
name: implement
description: Implement tasks from docs/task/*.md. Reads the task document, follows implementation steps, and updates status in TASKS.md. Use "/implement auto {task}" to auto-chain through the full pipeline. Use this skill whenever the user wants to start coding a planned task, build a feature, fix a bug, or kick off any implementation work.
---

# /implement - Implementation Agent

## Token Efficiency

| Mode | How it works | Token cost |
|------|-------------|------------|
| **Auto** | Orchestrator stays alive, spawns workers in background. Workers output a 4-line signal + write details to task doc. Orchestrator accumulates only signals (~50 tokens each). | Very low |
| **Manual + `/clear`** | Run each skill in a fresh session. Same token benefit, you stay in control. | Low |
| **Manual without `/clear`** | History compounds across all stages. | High (avoid) |

**Manual mode tip:** After each skill completes, run `/clear` before invoking the next one.

---

## Workflow

```
/implement [auto] [--worktree] {ID}
       ↓
0. Read LEARNINGS.md (if exists) — avoid known mistakes
1. Parse arguments: detect "auto" flag, "--model" override, "--worktree" flag, task ID
2. Resolve task ID → find docs/task/{ID}-{name}.md
3. Read task document — this is the ONLY codebase context needed
   └── Only read source files explicitly listed in task doc's "File Changes"
   └── Do NOT explore directories, grep broadly, or read files not in the plan
4. Conditional: Direct or Worktree
   ├─ Default (no --worktree): Commit to current branch
   │  └── All edits and commits happen directly in current directory
   │  └── User's working directory receives changes
   │
   └─ With --worktree: Enter worktree isolation
      └── EnterWorktree({ name: "task-{ID}" })
      └── Creates .claude/worktrees/task-{ID}/ on a new branch
      └── All edits, commits, and pushes happen inside the worktree
      └── User's working directory stays on main — never switches branch
5. If "auto" flag → set Automation: auto in task doc
6. Move task to "## In Progress" in TASKS.md
7. Invoke specialized skills if relevant (see Step 3 below)
8. Implement following task document steps
9. Commit with [task-{ID}] prefix for traceability
       ↓
┌─── Automation Mode? ───┐
│                        │
▼ Manual                 ▼ Auto
STOP (no push)           10. Push branch to origin
Update status to         11. Update status to "TESTING"
"TESTING" (keep local)   12. Enter Orchestrator Mode
Notify user ready for        (drives pipeline, stays alive)
/simplify
```

---

## Pre-Implementation Checklist

### 0. Read Project Learnings

- Read LEARNINGS.md entirely if it exists (~200 tokens, saves thousands)
- Focus on: Common Mistakes to Avoid, Established Coding Patterns, Architecture & Decisions, Tech Stack Notes
- The task doc contains all codebase research you need — only open source files explicitly listed in `## File Changes`; do NOT explore directories or read unlisted files

### 0.5. Parse Arguments

- `auto` → set automation mode
- `--model {haiku|sonnet|opus}` → model override (takes precedence over task doc recommendation)
- `--worktree` → enable worktree isolation (creates isolated branch)
- Natural language hints:
  - Model: `"use sonnet"`, `"force haiku"`, `"use opus"` → treat as model override
  - Worktree: `"use worktree"`, `"with worktree"`, `"isolated"`, `"in worktree"` → treat as `--worktree`
- Remaining arg → task ID

**Examples:**
```
/implement {ID}                              # Default: direct to current branch
/implement --worktree {ID}                   # Formal flag: use worktree isolation
/implement {ID} "use worktree"               # Natural language
/implement {ID} with worktree                # Natural language (no quotes)

/implement auto {ID}                         # Auto, direct to current branch (default)
/implement auto --worktree {ID}              # Auto with worktree isolation
/implement auto {ID} "use worktree"          # Auto, natural language

/implement --model sonnet {ID}               # Model override, direct to main
/implement --model sonnet --worktree {ID}    # Model override + worktree
/implement {ID} "use sonnet with worktree"   # Natural language, both overrides
```

**Argument resolution:**

1. **Model** (priority order):
   - `--model` flag or `"use X"` natural language → use this
   - `Recommended Model` field in task doc → use this
   - Default → `haiku`

2. **Worktree** (priority order):
   - `--worktree` flag or worktree-related natural language → use worktree isolation
   - Otherwise → default direct to current branch

**Acknowledge overrides in output:**
```
Model: sonnet (override — task recommended: haiku)
Worktree: enabled (isolated branch .claude/worktrees/task-{ID}/)
```

If `auto` detected, update task document header:
```markdown
> **Automation:** auto
```

### 1. Resolve Task ID

1. Read TASKS.md
2. Find row with matching ID in first column
3. Get task doc path from that row
4. Verify file exists in `docs/task/`

### 1.5. Worktree Isolation (Conditional)

**Default behavior — direct to current branch:**

- Commit directly to the current branch (wherever user is)
- No worktree isolation
- User's working directory changes are the final state
- Useful for: quick manual implementations, small fixes, direct branch work

**If `--worktree` flag present — use worktree isolation:**

```
EnterWorktree({ name: "task-{ID}" })
```

- Creates `.claude/worktrees/task-{ID}/` on a new branch
- User's working directory stays on `main` for the entire implementation
- Multiple agents can run in parallel with zero branch conflicts

**When to use `--worktree`:**
- Auto mode (recommended): Orchestrator isolated from user's branch, safer for full pipelines
- Multi-parallel tasks: Prevents branch conflicts when running multiple agents simultaneously
- Complex features: Isolated branch work, easier to abandon/reset if needed

**When to skip worktree (default):**
- Manual mode: User wants direct changes to review locally before pushing
- Quick fixes: Overhead not worth it for small tasks
- Working on a feature branch: Natural to commit directly

### 2. Read the Task Document — Then Stop Exploring

```
✅ ALWAYS READ:
   - docs/task/{ID}-{task-name}.md          (your primary source)
   - LEARNINGS.md                            (already done in Step 0)
   - Files listed in ## File Changes         (only these, nothing else)

❌ DO NOT READ unless the task doc explicitly names them:
   - package.json / tsconfig.json / .env
   - Any config file not in ## File Changes
   - Files in directories not mentioned in the task doc
   - Adjacent files "for context"

❌ DO NOT RUN:
   - find . -type f       (directory sweeps)
   - ls -R / ls -la       (directory listing)
   - cat on unlisted files
   - Any grep to "understand patterns" — patterns are in ## Code Context
```

If a file isn't in the task doc and you think you need it: check `## Code Context` first. If it genuinely isn't there, use a targeted `Grep` for the specific symbol — not a directory sweep.

Understand from the task doc:
- Task ID (for commit messages)
- **Recommended Model** — advisory, not a blocker; note if different but proceed
- Requirements, proposed solution, architecture
- File changes — your complete reading list
- Implementation steps — follow them in order

### 3. Invoke Specialized Skills

Before writing any code, check whether platform-specific skills are installed and relevant:

| Skill | Invoke When |
|-------|-------------|
| `/vercel-react-best-practices` | Task involves React, Next.js, or TypeScript |
| `/supabase-postgres-best-practices` | Task involves database queries, RLS, schema, or Supabase |

If a relevant skill is available, invoke it before writing code. If you realize partway through you skipped one, invoke it then review your code before continuing.

### 4. Update TASKS.md

Move task from "Planned" to "In Progress" with started date and status.

### 5. Verify Dependencies

- Required API endpoints exist
- Required types/interfaces exist
- Required packages installed
- No blocking tasks

---

## Commit Convention

**All commits must include the task ID prefix.**

```
[task-{ID}] {type}: {description}

{optional body}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

```bash
git commit -m "[task-1] feat: Add JWT authentication middleware

Implements token validation and refresh logic.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Completion Checklist

| Category | Checks |
|----------|--------|
| Code quality | Compiles without errors, no TypeScript errors, follows existing patterns |
| Functionality | All "Must Have" requirements implemented, happy/loading/error/empty states handled |
| Task document | Requirements checked off, deviations documented, notes for tester added |

---

## Update Status to TESTING

When implementation is complete:

**1. Update TASKS.md** — move to "Testing" section with test report pending.

**2. Update task document:**
```markdown
> **Status:** TESTING
> **Completed:** {Date}
> **Implementation Notes:** {Any important notes for tester}
```

**3. Update CLAUDE.md (Structural Changes)**

Scan what was built and update `CLAUDE.md` immediately — don't wait for `/document`.

| What you built | What to update in CLAUDE.md |
|----------------|------------------------------|
| New directory created | `## Project Structure` |
| File moved or renamed | `## Important File Locations` |
| Database migration added | `## Stack` (schema note) + `## Important File Locations` |
| New edge function / API route | `## Important File Locations` or `## Key Conventions` |
| New env variable required | `## Stack` (note the var and purpose) |
| New package installed | `## Stack` |
| New pattern established (used 2+ times) | `## Key Conventions` |
| Old pattern replaced | Remove old entry, add new one |

- Read existing CLAUDE.md before touching it — prune stale entries first
- Never just append — if a directory was renamed, update the old entry
- If none of the above apply (pure logic change, styling tweak) → skip

**4. Pre-Completion Verification:**
- React/Next.js task? → Invoked `/vercel-react-best-practices` (if installed)?
- Database/Supabase task? → Invoked `/supabase-postgres-best-practices` (if installed)?
- If skipped, invoke now and review code before proceeding.

**5. Route based on Automation Mode:**

Check `Automation: auto` field in task document.

---

### Manual Mode: No Push (Token Optimization)

```
Output: completion summary with:
  - Files changed locally
  - Commits made [task-{ID}]
  - Location info:
    * Default: current branch (files in user's working directory)
    * With --worktree: .claude/worktrees/task-{ID}/ (isolated branch)
  - Next step: /simplify {ID}

⚠️ IMPORTANT: Do NOT run git push
   Changes are committed locally (in current branch or worktree).
   User may review, edit, or run /simplify immediately.
```

Run `/clear` before invoking `/simplify` — it only needs the task doc, not this history.

**Default (direct):** Changes directly in user's working directory on current branch. Commit visible immediately.

**With --worktree:** Changes isolated in `.claude/worktrees/task-{ID}/`. User's main branch untouched.

---

### Auto Mode: Push & Enter Orchestrator

4. Push branch to origin:
```bash
git push -u origin {branch-name}
```

5. Enter Orchestrator Mode below.

---

## Orchestrator Mode

**Only entered when `Automation: auto` is set AND branch has been pushed to origin.** This session stays alive as the pipeline coordinator. Workers run in background, write results to the task document, and return a 4-line signal. You read the signal and decide what happens next — never the worker.

**Shared memory:** `docs/task/{ID}-{task-name}.md` — every worker reads and writes to it.

```
retry_count = 0
max_retries = 3
```

---

### Stage 1 — Quality Gate

Assign simplify worker:
```
Task({ subagent_type: "general-purpose", model: "sonnet", prompt: "/simplify {ID}", run_in_background: true })
```

**Wait for SIGNAL. Route:**

| Signal | Action |
|--------|--------|
| `STATUS: PASS` | Proceed to Stage 2 |
| `STATUS: FAIL` | retry_count++. If retry_count > max_retries → STOP, notify user. Otherwise: assign implement-fix worker (see Fix Worker below), then return to Stage 1 |

---

### Stage 2 — Testing

Assign test worker:
```
Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/test {ID}", run_in_background: true })
```

**Wait for SIGNAL. Route:**

| Signal | Action |
|--------|--------|
| `STATUS: PASS` | Proceed to Stage 3 |
| `STATUS: FAIL` | retry_count++. If retry_count > max_retries → STOP, notify user. Otherwise: assign implement-fix worker referencing `REFERENCE` from signal, then return to Stage 1 |

---

### Stage 3 — Documentation

Assign document worker:
```
Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/document {ID}", run_in_background: true })
```

Wait for `STATUS: DONE` signal → proceed to Stage 4.

---

### Stage 4 — Ship

Assign ship worker:
```
Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/ship {ID}", run_in_background: true })
```

Wait for `STATUS: DONE` signal → pipeline complete.

---

### Pipeline Complete

Report to user:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PIPELINE COMPLETE: #{ID} - {Task Title}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ implement   code written & committed
✓ simplify    quality gate passed
✓ test        {N} criteria verified
✓ document    feature doc created
✓ ship        PR ready for review

PR: {url from ship signal}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Fix Worker (when simplify or test fails)

```
Task({
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: "/implement {ID} — Fix required. See REFERENCE path in task doc for details.",
  run_in_background: true
})
```

The worker reads the task document fresh — all context is in `## Implementation Notes` or the test report path. Pass only the reference path, not the full failure details.

---

### Stop Conditions

Stop the pipeline and notify the user when:
- `retry_count > max_retries` (3 cycles without PASS)
- Any worker returns `STATUS: BLOCKED`
- Ship worker returns a build/lint failure

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PIPELINE STOPPED: #{ID} - {Task Title}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Stopped at: {stage}
Reason: {signal summary}
Details: {REFERENCE path from signal}

Action needed from you before pipeline can continue.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Specialized Skills (Install Separately)

- `vercel-react-best-practices` — install from [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills); invoke for React/Next.js/TypeScript code
- `supabase-postgres-best-practices` — install from [supabase/agent-skills](https://github.com/supabase/agent-skills); invoke for database queries, RLS, schema, Supabase code
