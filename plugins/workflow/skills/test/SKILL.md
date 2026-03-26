---
name: test
description: Test web implementations using Playwright MCP for E2E testing. Creates test reports in docs/testing/*.md.
---

# /test - Web E2E Testing Agent

## Prerequisites: Playwright MCP Setup

- No npm install needed — Claude Code handles it automatically
- Create `.mcp.json` in project root (not in `.claude/` or subdirectories):
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
- Restart Claude Code after adding/modifying `.mcp.json`
- Verify MCP active: look for `mcp__playwright__browser_*` tools

**CI Mode requirements** (no MCP needed):
```bash
npm install -D @playwright/test
npx playwright install
```

---

## CRITICAL: Mode Detection (Execute FIRST)

```
STEP 1: Parse for --ci flag
           │
    ┌──────┴──────┐
    ▼             ▼
Has --ci?     No --ci?
    │             │
    ▼             ▼
MODE = CI     STEP 2: Check for mcp__playwright__browser_* tools
              │
     ┌────────┴────────┐
     ▼                 ▼
Available?         NOT Available?
     │                 │
     ▼                 ▼
MODE = Interactive   Check Automation field
                     │
            ┌────────┴────────┐
            ▼                 ▼
      Automation: auto    Manual mode
            │                 │
            ▼                 ▼
      Fall back to CI     STOP — display
      Notify user         prereqs, do NOT
                          fall back or proceed
```

**ABSOLUTE RULES:**
1. No `--ci` + No Playwright MCP + Manual mode = **STOP**. Do not proceed. Do not fall back.
2. No `--ci` + No Playwright MCP + Auto mode = **Fall back to CI silently**. Notify user and continue.
3. No `--ci` + Playwright MCP available = **Interactive mode ONLY**. Never create test scripts.
4. `--ci` present = **CI mode**. Playwright MCP not required.

---

## CRITICAL: Mode-Specific Rules

### Interactive Mode — What NOT To Do

- Do NOT create `.ts`, `.js`, or `.spec` test files
- Do NOT create `TEST_REPORT.md`, `TEST_SUMMARY.txt` in project root
- Do NOT create `test-screenshots/` or similar folders in project root
- Do NOT run `npx playwright test`
- Do NOT spawn subagents to run headless tests
- Use `mcp__playwright__browser_*` tools directly in conversation
- Write final report ONLY to `docs/testing/{task-name}.md`

### CI Mode — Allowed Actions

- Create test scripts in `tests/e2e/{task-name}/` (NOT project root)
- Run `npx playwright test` for headless execution
- Spawn subagents for parallel test execution
- Write final report to `docs/testing/{task-name}.md`
- Cleanup all temporary artifacts after tests
- Ask user if they want to keep test scripts for CI pipeline

---

## Workflow

**Always execute Mode Detection FIRST.**

### Interactive Mode (Default)

```
/test {task-name}
       ↓
1. Read task document: ## Implementation Notes (primary), ## Acceptance Criteria (test cases)
2. Build test plan mapping each acceptance criterion to a Playwright action
3. Execute tests via mcp__playwright__browser_* tools directly
   ⚠️ DO NOT create test script files
4. Write report to docs/testing/{ID}-{task-name}.md using: docs/templates/test-report.md
5. Update TASKS.md
       ↓
Route on automation mode (see below)
```

### CI Mode (`--ci` flag)

```
/test --ci {task-name}
       ↓
1. Read task document for requirements
2. Create test directory: tests/e2e/{task-name}/
3. Generate Playwright test script(s)
4. Run: npx playwright test
5. Capture results
6. Write report to docs/testing/{ID}-{task-name}.md using: docs/templates/test-report.md
7. Cleanup temporary artifacts (keep test scripts by default)
8. Update TASKS.md
       ↓
Route on automation mode (see below)
```

```
/test --ci --cleanup {task-name}
       ↓
Same as above, but ALSO deletes tests/e2e/{task-name}/
```

---

## CI Mode: Directory Structure (CRITICAL)

```
project-root/
├── tests/
│   └── e2e/
│       └── {task-name}/           ← Test scripts go HERE (*.spec.ts)
│           ├── {task-name}.spec.ts
│           └── helpers.ts (if needed)
│
├── docs/
│   └── testing/
│       └── {task-name}.md         ← ONLY the final report goes HERE
│
└── (temporary — ALWAYS DELETE after tests)
    ├── TEST_REPORT.md, TEST_CASES.md, TEST_SUMMARY.txt
    ├── test_results.md, test_execution_summary.md
    ├── TEST_REPORT_INDEX.md, playwright_mcp_test_log.md
    ├── test-screenshots/, test-results/, playwright-report/
```

**ONLY allowed in `docs/testing/`:**
- `docs/testing/{task-name}.md` (final test report, one per task)
- `docs/testing/README.md` (if project has one)

---

## CI Mode: Artifact Management

| Artifact | Location | `--ci` | `--ci --cleanup` |
|----------|----------|--------|------------------|
| Final test report | `docs/testing/{task-name}.md` | Keep | Keep |
| Test scripts | `tests/e2e/{task-name}/` | Keep | Delete |
| `TEST_CASES.md` | anywhere | Delete | Delete |
| `test_results.md` | anywhere | Delete | Delete |
| `test_execution_summary.md` | anywhere | Delete | Delete |
| `TEST_REPORT_INDEX.md` | anywhere | Delete | Delete |
| `playwright_mcp_test_log.md` | anywhere | Delete | Delete |
| `TEST_SUMMARY.txt` | anywhere | Delete | Delete |
| `test-screenshots/` | anywhere | Delete | Delete |
| `playwright-report/` | anywhere | Delete | Delete |
| `test-results/` | anywhere | Delete | Delete |

**Cleanup commands (run at end of every CI test):**
```bash
rm -f TEST_REPORT.md TEST_SUMMARY.txt TEST_CASES.md
rm -f test_results.md test_execution_summary.md TEST_REPORT_INDEX.md
rm -f playwright_mcp_test_log.md
rm -rf test-screenshots/ test-results/ playwright-report/
rm -f test-*.ts test-*.js *.spec.ts
rm -f docs/testing/TEST_CASES.md docs/testing/test_results.md
rm -f docs/testing/test_execution_summary.md docs/testing/TEST_REPORT_INDEX.md
rm -f docs/testing/playwright_mcp_test_log.md docs/testing/TEST_SUMMARY.txt
rm -f docs/testing/*.spec.ts
```

---

## Pre-Testing Setup

Read task document in this priority order:
1. `## Implementation Notes` — how to access and test what was built (URL, credentials, setup)
2. `## Acceptance Criteria` — Given/When/Then scenarios to verify (your test cases)
3. If `## Implementation Notes` missing, fall back to `## Requirements` and `## File Changes`

Do NOT scan the codebase or infer test scenarios.

**Map each criterion to a Playwright action:**
```
Given I am on `/auth/login`          → navigate to URL
When I submit valid credentials      → fill form + click submit
Then I am redirected to `/dashboard` → assert URL changed
```

Execute: happy path → error states → edge cases.

---

## Testing with Playwright MCP

```
mcp__playwright__browser_navigate    - Go to URL
mcp__playwright__browser_snapshot    - Capture accessibility snapshot
mcp__playwright__browser_click       - Click elements
mcp__playwright__browser_type        - Type text
mcp__playwright__browser_fill_form   - Fill form fields
mcp__playwright__browser_take_screenshot - Capture screenshot
mcp__playwright__browser_console_messages - Check console
mcp__playwright__browser_network_requests - Check network
mcp__playwright__browser_wait_for    - Wait for conditions
```

**Test execution pattern:**
```typescript
// 1. Navigate
mcp__playwright__browser_navigate({ url: "http://localhost:3000/path" })
// 2. Snapshot to understand structure
mcp__playwright__browser_snapshot()
// 3. Interact (use ref from snapshot)
mcp__playwright__browser_click({ ref: "button[Submit]", element: "Submit button" })
// 4. Verify
mcp__playwright__browser_snapshot()
// 5. Check errors
mcp__playwright__browser_console_messages({ level: "error" })
```

**Responsive testing:**
```typescript
mcp__playwright__browser_resize({ width: 390, height: 844 }) // iPhone 14 Pro
mcp__playwright__browser_resize({ width: 768, height: 1024 }) // iPad
```

---

## Write Test Report

Write report to `docs/testing/{ID}-{task-name}.md` using: `docs/templates/test-report.md`

---

## Update TASKS.md

- **PASS:** Move to "Approved" section with status "PASS - Awaiting approval"
- **FAIL:** Keep in "Testing" with status "FAIL - See report"

---

## Auto Mode Behavior

When task document has `Automation: auto`, this skill is a **worker** — output SIGNAL and exit. Do not spawn next skill.

### On PASS
```
≡ SIGNAL
STAGE: test
STATUS: PASS
TASK: {ID}
SUMMARY: {N}/{N} acceptance criteria verified
REFERENCE: docs/testing/{ID}-{task-name}.md
≡ END
```

**Manual mode:** Run `/clear` then `/document {ID}`.

### On FAIL

Write failure block to task document `## Implementation Notes`:
```markdown
### Test failure (attempt {N})
- **Failed criteria:** {list which acceptance criteria failed}
- **Root cause:** {specific observable failure — URL, element, response}
- **Files to check:** {files most likely involved}
```

```
≡ SIGNAL
STAGE: test
STATUS: FAIL
TASK: {ID}
SUMMARY: {N} criteria failed — details in test report
REFERENCE: docs/testing/{ID}-{task-name}.md
≡ END
```

**Manual mode:** Run `/clear` then `/implement {ID}` → `/simplify {ID}` → `/test {ID}`.

---

## Manual Testing Guidance

For features that can't be fully automated, document in report:

| Test | Instructions | Expected Result |
|------|--------------|-----------------|
| Visual accuracy | Compare to design | Matches mockup |
| Animation | Trigger X action | Smooth 60fps |

**Manual mode handoff:**
- PASS: notify user, report at `docs/testing/{ID}-{task-name}.md`, next step `/document {ID}`
- FAIL: notify user, report at `docs/testing/{ID}-{task-name}.md`, next step `/implement {ID}` then re-run `/test {ID}`
