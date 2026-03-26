# {Task Title}

> **ID:** {number}
> **Status:** PLANNED
> **Priority:** HIGH | MEDIUM | LOW
> **Type:** feature | bugfix | enhancement | documentation | chore
> **Version Impact:** minor | patch | major
> **Recommended Model:** haiku | sonnet
> **Created:** {Date}
> **Platform:** Web
> **Automation:** manual | auto

## Overview

{2-3 sentence description of what we're building and why}

## Development Approach

**Methodology:** {TDD | CDD | Standard}
**Rationale:** {1 sentence — why this fits the task type}

| Task type | Recommended |
|-----------|-------------|
| New UI feature | CDD — build atoms before composites |
| Business logic / utilities | TDD — pure functions, testable units |
| API integration | Standard with BDD acceptance criteria |
| Bug fix | Standard — targeted, minimal change |

## Requirements

### Must Have
- [ ] Requirement 1
- [ ] Requirement 2

### Nice to Have
- [ ] Optional requirement

## Current State

{Description of how things work now, if applicable}

## Proposed Solution

{Description of the implementation approach}

### Architecture

{High-level design decisions}

### File Changes

| Action | File | Description |
|--------|------|-------------|
| CREATE | `path/to/new.tsx` | New component for X |
| MODIFY | `path/to/existing.tsx` | Add Y functionality |
| DELETE | `path/to/old.tsx` | No longer needed |

## Code Context

> Embedded by /task during research so /implement never re-reads these files.
> Paste the relevant sections — not entire files. Focus on what will change.

### `path/to/file-being-modified.tsx` (MODIFY)

```typescript
// Current implementation of the function/component being changed
// Paste only the relevant section, not the whole file
```

### Import Patterns

```typescript
// How imports are done in files adjacent to the ones being changed
// e.g., how auth, routing, or data-fetching is imported in this area
```

### Related Patterns (if applicable)

```typescript
// Existing similar implementation to follow as a model
// e.g., "this new component should follow the same pattern as ComponentX"
```

## Implementation Steps

### Step 1: {Title}
{Detailed instructions — reference specific lines/functions from Code Context above}

### Step 2: {Title}
{Detailed instructions}

## Acceptance Criteria

Write these as specific, observable behaviors — not requirements. The test agent executes these directly via Playwright, so each criterion must be verifiable by looking at the UI or checking a response.

### Happy path
- [ ] Given {starting state}, when {action}, then {observable outcome}
- [ ] Given I am on `/auth/login`, when I submit valid credentials, then I am redirected to `/dashboard`

### Error states
- [ ] Given {condition}, when {action}, then {specific error message or behavior}
- [ ] Given I submit an invalid password, then an inline error appears below the form (not a page alert)

### Edge cases
- [ ] {Specific scenario that could break things}

### Test setup
- **URL:** {entry point for testing}
- **Test credentials:** {email/password if auth, or "N/A"}
- **Setup required:** {seed data, env vars, migrations — or "None"}

## Dependencies

- Required packages: {list any new deps}
- Required APIs: {list endpoints needed}
- Blocked by: {any dependencies on other tasks}

## Notes for Implementation Agent

{Any important context the /implement agent needs to know}

## Related

- Similar feature: [link to docs]
- Design reference: [link if applicable]
