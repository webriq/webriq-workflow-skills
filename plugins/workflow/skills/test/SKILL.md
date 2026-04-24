---
name: test
description: Use this skill after implementation passes the quality gate and the user wants verification. Runs capability-based checks from the task document, including unit, integration, browser, API, CLI, CI, or manual checks, and writes docs/testing/*.md.
---

# test - Verification Stage

## Invariants

- Verify against the task document acceptance criteria.
- Use the best available project verification path.
- Do not require one specific browser automation tool in core workflow.
- Always write one final report to `docs/testing/`.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `test`.

## Workflow

1. Read `AGENTS.md`.
2. Resolve the task ID and read `docs/task/{ID}-{task-name}.md`.
3. Confirm the task reached the quality gate.
4. Build a verification plan from `Acceptance Criteria` and `Verification Approach`.
5. Prefer existing project commands and CI-compatible checks.
6. Run safe verification commands.
7. Use browser automation or manual checks only when required by the task.
8. Write `docs/testing/{ID}-{task-name}.md` using `docs/templates/test-report.md`.
9. Move the task to `Approved` only after tests pass and the user approves, or leave it in `Testing` if fixes are needed.

## Verification Priority

Use the narrowest reliable check first:

1. Unit tests for pure logic.
2. Integration tests for API, database, service, or cross-module behavior.
3. Browser or end-to-end checks for user-visible workflows.
4. Static checks such as typecheck, lint, or build where they are part of project quality.
5. Manual inspection only when automation is unavailable or not cost-effective.

If no viable verification path exists, write a BLOCKED report explaining what is missing.

## Report Rules

- Write the final report only under `docs/testing/`.
- Do not scatter temporary reports in the repo root.
- Record commands exactly as run.
- Record skipped checks with reasons.
- Include enough evidence for the implementation stage to fix failures without session history.

## Handoff

PASS:

```text
Verification passed for task {ID}
Report: docs/testing/{ID}-{task-name}.md
Next stage after user approval: document {ID}
```

FAIL or BLOCKED:

```text
Verification failed or blocked for task {ID}
Report: docs/testing/{ID}-{task-name}.md
Return to implementation with the report context.
```
