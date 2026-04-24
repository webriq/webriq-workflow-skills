# {Task Title}

> **ID:** {number}
> **Status:** PLANNED
> **Priority:** HIGH | MEDIUM | LOW
> **Type:** feature | bugfix | enhancement | documentation | chore
> **Version Impact:** major | minor | patch
> **Recommended Tier:** fast | balanced | deep
> **Created:** {Date}
> **Platform:** {web | api | cli | docs | mixed}
> **Automation:** manual | auto

## Overview

{2-3 sentence description of what is being changed and why.}

## Development Approach

**Methodology:** TDD | CDD | Standard
**Rationale:** {Why this approach fits the task.}

## Requirements

### Must Have

- [ ] {Required behavior}
- [ ] {Required behavior}

### Nice to Have

- [ ] {Optional behavior}

## Out of Scope

- {Explicitly excluded behavior or files}
- {What must not change}

## Current State

{How the relevant system works now. Include only context needed for implementation.}

## Proposed Solution

{Implementation approach and key decisions.}

### File Changes

| Action | File | Description |
|--------|------|-------------|
| CREATE | `path/to/new-file` | {Why this file is needed} |
| MODIFY | `path/to/existing-file` | {What changes here} |
| DELETE | `path/to/old-file` | {Why deletion is safe} |

## Code Context

> Embedded by the `task` stage during targeted research. Include relevant excerpts only, not full files.

### `path/to/file`

```text
{Relevant excerpt or summary}
```

## Implementation Steps

### Step 1: {Title}

{Specific instructions.}

### Step 2: {Title}

{Specific instructions.}

## Acceptance Criteria

Write criteria as observable behavior that can be verified by tests, commands, browser automation, API calls, or document inspection.

### Happy Path

- [ ] Given {state}, when {action}, then {observable result}

### Error States

- [ ] Given {condition}, when {action}, then {error or recovery behavior}

### Edge Cases

- [ ] {Boundary case}

## Verification Approach

### Commands

```bash
{project-specific command, e.g. pnpm test}
{project-specific command, e.g. pnpm lint}
```

### Manual Or Browser Checks

- {Manual or browser automation steps, if needed}

### Test Data And Setup

- **URL or entry point:** {N/A if not applicable}
- **Credentials:** {N/A or test credentials}
- **Seed data / env vars / migrations:** {N/A or details}

## Compatibility Touchpoints

- **Packaging / install surface:** {Does this affect skills, manifests, package metadata, or install docs?}
- **Docs:** {README, quick reference, guides, or task templates to update}
- **Adapters:** {Claude, Codex, Gemini, or other runtime notes to update}

## Dependencies

- Required packages: {None or list}
- Required APIs: {None or list}
- Blocked by: {None or task/reference}

## Notes For Implementation Agent

{Critical constraints, gotchas, and tier rationale.}
