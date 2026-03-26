# Test Report: {Task Name}

> **Task ID:** {ID}
> **Status:** PASS | FAIL | PARTIAL
> **Tested:** {Date}
> **Task Doc:** [link](../task/{ID}-{task-name}.md)

## Summary

{1-2 sentence summary of test results}

## Test Environment

- Platform: Web
- Browser: Chromium (Playwright)
- Base URL: http://localhost:3000
- Viewport: {dimensions if relevant}

## Test Results

### Requirement Tests

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | {Requirement from task doc} | PASS/FAIL | {Details} |

### Functional Tests

#### Test 1: {Test Name}
**Steps:**
1. Navigate to {page}
2. Click {element}
3. Verify {expected result}

**Result:** PASS/FAIL
**Evidence:** {Screenshot or description}

### Edge Cases

| Case | Result | Notes |
|------|--------|-------|
| Empty state | PASS/FAIL | {Details} |
| Error handling | PASS/FAIL | {Details} |
| Loading state | PASS/FAIL | {Details} |

### Console Errors

```
{Any console errors found, or "None"}
```

### Network Issues

```
{Any failed requests, or "None"}
```

## Issues Found

### Issue 1: {Title}
**Severity:** Critical | Major | Minor
**Description:** {What's wrong}
**Steps to Reproduce:**
1. Step 1
2. Step 2
**Expected:** {What should happen}
**Actual:** {What actually happens}

## Manual Testing Required

| Test | Instructions | Expected Result |
|------|--------------|-----------------|

## Recommendations

{Any suggestions for fixes or improvements}

## Verdict

**PASS** - All requirements met, ready for documentation
OR
**FAIL** - Issues found, needs fixes (see Issues section)
OR
**PARTIAL** - Core functionality works, minor issues noted
