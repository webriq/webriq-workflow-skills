---
name: simplify
description: Code quality and deviation gate between /implement and /test. Reads the task document and changed files, validates coding standards, classifies deviations (minor/medium/major), and decides whether implementation is ready for testing. Runs automatically in the auto-chain between implement and test. Also invoke manually after any implementation to catch issues before wasting a test run.
model: sonnet
---

# /simplify - Quality Gate Agent

> **Model:** sonnet (reasoning needed to evaluate code quality and detect deviations)

## Command Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--help` | `-h` | Show available commands and options |
| `--version` | `-v` | Show workflow skills version |

### Flag Handling

**On `-h` or `--help`:**
```
/simplify - Quality Gate Agent

Usage:
  /simplify {ID}                     Review implementation for a task
  /simplify -h, --help               Show this help message
  /simplify -v, --version            Show version

Arguments:
  {ID}    Task ID (number) or task filename (e.g., 001-auth-jwt)

Checks:
  - Coding standards (no any types, guard clauses, naming, etc.)
  - Methodology compliance (TDD/CDD/SOLID if specified in task doc)
  - Deviation classification (minor / medium / major)

Result:
  PASS → chains to /test
  FAIL → reports issues, blocks until resolved

Examples:
  /simplify 1                        # Review task #1 implementation
  /simplify 001-auth-jwt             # Using task filename

Next: /test {ID}
```

**On `-v` or `--version`:**
```
Workflow Skills v1.5.0
https://github.com/eljun/workflow-skills
```

---

## Workflow

```
/simplify {ID}
       ↓
1. Resolve task ID → read task document
2. Identify changed files (Implementation Notes or git diff)
3. Read changed files
4. Run coding standards checklist
5. Check methodology compliance (if specified in task doc)
6. Classify deviations from the plan
7. Write Implementation Notes to task document
       ↓
┌─── Result ───────────────┐
│                          │
▼ PASS                     ▼ FAIL
Invoke /test {ID}          Report to human, stop
```

---

## Step 1: Read Context

Resolve task ID and read the task document:
```
docs/task/{ID}-{task-name}.md
```

Focus on:
- `## Development Approach` — methodology to validate against
- `## Acceptance Criteria` — what was supposed to be built
- `## File Changes` — which files were planned to change

Then identify what was actually changed:
```bash
git diff --name-only main...HEAD
```

Read only the changed files — not the entire codebase.

---

## Step 2: Coding Standards

Review each changed file. These apply regardless of framework or methodology.

### TypeScript / JavaScript
- [ ] No `any` types — use proper types, generics, or `unknown`
- [ ] No unused variables or imports
- [ ] Named constants for magic strings/numbers (no hardcoded values inline)
- [ ] Early returns / guard clauses — avoid nesting deeper than 3 levels
- [ ] Functions do one thing (single responsibility)
- [ ] Functions fit in ~30 lines (soft limit — break if it forces artificial splits)
- [ ] Descriptive names — functions read as sentences (`getUserByEmail`, not `getUser`)
- [ ] No commented-out code left behind
- [ ] No `console.log` in production paths

### React / UI (if applicable)
- [ ] Components have explicit Props types
- [ ] Hooks called before any early returns
- [ ] Loading, error, and empty states handled
- [ ] No unnecessary re-render triggers (inline anonymous functions in JSX)

### General
- [ ] No deep nesting — prefer flat logic and early exits
- [ ] No duplicate logic — if the same pattern appears 3+ times, it should be extracted
- [ ] Immutable patterns where possible (avoid mutating arrays/objects in place)

---

## Step 3: Methodology Compliance

If the task document has a `## Development Approach` section, validate against it:

**TDD:**
- Does the implementation directly satisfy each acceptance criterion?
- Are there test files included in the changed set (if TDD was specified)?

**CDD (Component-Driven):**
- Were atomic components built before composite ones?
- Are components independently usable, not tightly coupled to page context?

**SOLID:**
- Single Responsibility: do files/functions have one clear job?
- Dependency Inversion: are dependencies injected rather than hardcoded?

If no methodology was specified, skip this step.

---

## Step 4: Deviation Classification

Compare the implementation against the task document's `## Acceptance Criteria` and `## File Changes`.

### Minor deviation
- Different approach, same outcome
- Extra helper extracted that wasn't in the plan
- **Action:** Document in Implementation Notes, proceed to PASS

### Medium deviation
- A plan assumption was wrong (API shape differs, component didn't exist)
- Scope slightly larger or smaller but requirements still met
- **Action:** Document clearly, flag to human in result summary, still PASS (reviewed in PR)

### Major deviation
- Core approach is unworkable — implementation cannot meet acceptance criteria
- Requirements were misunderstood — what was built is fundamentally different
- A blocking dependency is missing
- **Action:** STOP. Write deviation summary. Do NOT chain to /test.

---

## Step 5: Write Implementation Notes

Write (or update) this section in the task document. This is the primary context `/test` uses — so be specific and concrete:

```markdown
## Implementation Notes

> **Simplify Review:** PASS | FAIL
> **Reviewed:** {Date}

### What was built
{Concrete description of behavior — what the user/system can now do, not what files changed}

### How to access for testing
- URL: {if applicable}
- Entry point: {button, page, API endpoint}
- Test credentials: {if auth involved}
- Setup required: {seed data, env vars, migrations, etc.}

### Deviations from plan
{None | Description of minor/medium deviations found}

### Standards check
{Pass | List any issues found and how they were resolved}
```

---

## Step 6: Result

### PASS

```
Quality review: PASS for #{ID} - {Task Title}

Standards: ✓
Deviations: {None | Minor — documented}
Methodology: {N/A | Compliant}

Implementation Notes written to task doc.

[AUTO] Spawning /test...
```

Use Task tool: `Task({ subagent_type: "general-purpose", model: "haiku", prompt: "/test {ID}" })`

### FAIL — Standards Issues

```
Quality review: FAIL for #{ID} - {Task Title}

Issues to fix before testing:
1. {file}: {issue}
2. {file}: {issue}

Run /implement {ID} to fix, then /simplify {ID} again.
```

Do NOT chain to /test.

### FAIL — Major Deviation

```
Quality review: BLOCKED for #{ID} - {Task Title}

Major deviation:
{What was planned vs what was built}

Impact: {Why this prevents meaningful testing}

Options:
1. Re-plan  → /task revise {ID}
2. Continue → tell me to proceed and I'll document the deviation
3. Abandon  → I'll mark the task as blocked in TASKS.md
```

---

## Auto Mode Behavior

When task document has `Automation: auto`:
- PASS → automatically spawns `/test {ID}` with haiku
- FAIL (standards) → stops, notifies user, cannot auto-fix
- FAIL (major deviation) → stops, presents options, awaits human decision

---

## Retry Context

When invoked as part of a retry cycle (after `/test` failed and `/implement` ran a fix), read the previous test report:
```
docs/testing/{ID}-{task-name}.md
```

Before proceeding, verify the fix actually addresses the specific failure:

```markdown
### Retry context (attempt {N})
- Previous failure: {summary of what /test reported}
- Fix applied: {what /implement changed}
- Fix verified: Yes | Partial | No
```

If the fix does not address the previous failure, FAIL immediately — this prevents burning another test run on the same issue.

---

## Related Skills

| Skill | When to Use |
|-------|-------------|
| `/implement` | If fixes needed — go back, fix, re-run /simplify |
| `/test` | After PASS — automatically chained in auto mode |
| `/task` | If major deviation — revise the plan |
