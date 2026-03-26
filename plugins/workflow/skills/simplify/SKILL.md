---
name: simplify
description: Code quality and deviation gate between /implement and /test. Reads the task document and changed files, validates coding standards, classifies deviations (minor/medium/major), and decides whether implementation is ready for testing. Runs automatically in the auto-chain between implement and test. Also invoke manually after any implementation to catch issues before wasting a test run.
---

# /simplify - Quality Gate Agent

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
Output SIGNAL, exit        Report issues, output SIGNAL, exit
```

---

## Step 1: Read Context

- Resolve task ID → read `docs/task/{ID}-{task-name}.md`
- Focus on: `## Development Approach`, `## Acceptance Criteria`, `## File Changes`
- Identify actually changed files: `git diff --name-only main...HEAD`
- Read only the changed files — not the entire codebase

---

## Step 2: Coding Standards

Review each changed file:

### TypeScript / JavaScript
- [ ] No `any` types — use proper types, generics, or `unknown`
- [ ] No unused variables or imports
- [ ] Named constants for magic strings/numbers
- [ ] Early returns / guard clauses — avoid nesting deeper than 3 levels
- [ ] Functions do one thing (single responsibility)
- [ ] Functions fit in ~30 lines (soft limit)
- [ ] Descriptive names — functions read as sentences (`getUserByEmail`, not `getUser`)
- [ ] No commented-out code
- [ ] No `console.log` in production paths

### React / UI (if applicable)
- [ ] Components have explicit Props types
- [ ] Hooks called before any early returns
- [ ] Loading, error, and empty states handled
- [ ] No unnecessary re-render triggers (inline anonymous functions in JSX)

### General
- [ ] No deep nesting — prefer flat logic and early exits
- [ ] No duplicate logic — if same pattern appears 3+ times, extract it
- [ ] Immutable patterns where possible

---

## Step 3: Methodology Compliance

If task doc has `## Development Approach`, validate against it:

- **TDD:** Does implementation satisfy each acceptance criterion? Test files included if TDD specified?
- **CDD:** Atomic components built before composites? Components independently usable?
- **SOLID:** Single Responsibility — files/functions have one clear job? Dependency Inversion — dependencies injected, not hardcoded?

If no methodology specified, skip this step.

---

## Step 4: Deviation Classification

Compare implementation against `## Acceptance Criteria` and `## File Changes`:

| Severity | Definition | Action |
|----------|-----------|--------|
| Minor | Different approach, same outcome; extra helper extracted | Document in Implementation Notes, PASS |
| Medium | Plan assumption wrong (API shape differs, component didn't exist); scope slightly off but requirements met | Document clearly, flag in summary, still PASS |
| Major | Core approach unworkable; requirements misunderstood; blocking dependency missing | STOP — write deviation summary, do NOT chain to /test |

---

## Step 5: Write Implementation Notes

Write (or update) this section in the task document:

```markdown
## Implementation Notes

> **Simplify Review:** PASS | FAIL
> **Reviewed:** {Date}

### What was built
{Concrete description of behavior — what the user/system can now do}

### How to access for testing
- URL: {if applicable}
- Entry point: {button, page, API endpoint}
- Test credentials: {if auth involved}
- Setup required: {seed data, env vars, migrations}

### Deviations from plan
{None | Description of minor/medium deviations found}

### Standards check
{Pass | List any issues found and how they were resolved}
```

---

## Step 6: Result

### PASS

**Auto mode — output SIGNAL and exit:**
```
≡ SIGNAL
STAGE: simplify
STATUS: PASS
TASK: {ID}
SUMMARY: {deviations summary | "No deviations"}
REFERENCE: docs/task/{ID}-{task-name}.md ## Implementation Notes
≡ END
```

**Manual mode:** Run `/clear` before invoking `/test {ID}`.

### FAIL — Standards Issues

Issues written to task doc `## Implementation Notes`.

**Auto mode — output SIGNAL and exit:**
```
≡ SIGNAL
STAGE: simplify
STATUS: FAIL
TASK: {ID}
SUMMARY: {N} standards issues found
REFERENCE: docs/task/{ID}-{task-name}.md ## Implementation Notes
≡ END
```

**Manual mode:** Fix with `/implement {ID}`, then re-run `/simplify {ID}`.

### FAIL — Major Deviation

**Auto mode — output SIGNAL and exit:**
```
≡ SIGNAL
STAGE: simplify
STATUS: BLOCKED
TASK: {ID}
SUMMARY: Major deviation — {one sentence}
REFERENCE: docs/task/{ID}-{task-name}.md ## Implementation Notes
≡ END
```

**Manual mode:** Present options — re-plan via `/task`, continue with documented deviation, or mark blocked in TASKS.md.

---

## Auto Mode Behavior

When task document has `Automation: auto`, this skill is a **worker** — it does not chain to the next stage. The orchestrator (`/implement auto`) drives all routing.

- PASS → output SIGNAL, exit
- FAIL (standards) → write issues to task doc, output SIGNAL, exit
- FAIL (major deviation) → output BLOCKED SIGNAL, exit

---

## Retry Context

When invoked as part of a retry cycle, read the previous test report: `docs/testing/{ID}-{task-name}.md`

Verify the fix actually addresses the specific failure before proceeding:

```markdown
### Retry context (attempt {N})
- Previous failure: {summary of what /test reported}
- Fix applied: {what /implement changed}
- Fix verified: Yes | Partial | No
```

If the fix does not address the previous failure, FAIL immediately.
