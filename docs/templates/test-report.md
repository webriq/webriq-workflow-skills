# Test Report: {Task Name}

> **Task ID:** {ID}
> **Status:** PASS | FAIL | PARTIAL | BLOCKED
> **Tested:** {Date}
> **Task Doc:** [docs/task/{ID}-{task-name}.md](../task/{ID}-{task-name}.md)

## Summary

{1-2 sentence result summary.}

## Test Environment

- Platform: {web | api | cli | docs | mixed}
- Runtime: {local | CI | staging | other}
- Tools: {unit test runner, integration test runner, browser automation, manual inspection}
- Base URL or entry point: {N/A or value}

## Verification Commands

| Command | Result | Notes |
|---------|--------|-------|
| `{command}` | PASS/FAIL/SKIPPED | {Details} |

## Acceptance Criteria Results

| # | Criterion | Status | Evidence |
|---|-----------|--------|----------|
| 1 | {Criterion from task doc} | PASS/FAIL | {Command output, observation, screenshot path, or notes} |

## Functional Checks

### Check 1: {Name}

**Steps:**

1. {Step}
2. {Step}
3. {Step}

**Result:** PASS | FAIL | SKIPPED
**Evidence:** {Description or artifact path}

## Issues Found

### Issue 1: {Title}

**Severity:** Critical | Major | Minor
**Description:** {What is wrong}
**Steps to reproduce:**

1. {Step}
2. {Step}

**Expected:** {Expected behavior}
**Actual:** {Actual behavior}

## Manual Testing Required

| Test | Instructions | Expected Result |
|------|--------------|-----------------|
| {Name} | {Steps} | {Expected result} |

## Deferred Or Blocked Checks

| Check | Reason | Follow-up |
|-------|--------|-----------|
| {Check} | {Why it was not run} | {What should happen next} |

## Verdict

**PASS** - All required checks passed.

**FAIL** - Blocking issues found; return to implementation.

**PARTIAL** - Core behavior works, but non-blocking issues or deferred checks remain.

**BLOCKED** - Required verification could not run.
