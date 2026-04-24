---
name: implement
description: Use this skill when an approved task document already exists and the user wants code or content changes implemented. Reads docs/task/*.md, follows the implementation steps, updates TASKS.md, and can run the approved auto chain from implementation onward.
---

# implement - Implementation Stage

## Invariants

- Require an approved task document before changing implementation files.
- Treat the task document as the primary scope contract.
- Do not broaden scope without user approval.
- Do not rely on provider-specific model switch commands.
- `implement auto` is the only automation entry point.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `implement`.

## Argument Handling

- `{ID}` or `{task-name}` identifies the task document.
- `auto` enables the implementation-to-ship automation chain after implementation.
- `--tier fast|balanced|deep` may override the task document recommendation only if the active runtime supports tier routing.

If a user asks for provider-specific model names, treat them as adapter-level preferences rather than core workflow semantics.

## Workflow

1. Read `AGENTS.md`.
2. Read `LEARNINGS.md` if it exists.
3. Resolve the task ID from `TASKS.md` or direct task filename.
4. Read `docs/task/{ID}-{task-name}.md`.
5. Verify the task has been reviewed and approved for implementation. If approval is unclear, stop and ask.
6. Read only files listed in the task document unless a missing dependency blocks implementation.
7. Move the task to `In Progress` in `TASKS.md`.
8. Apply the implementation steps.
9. Run relevant verification commands from the task document when safe.
10. Append `Implementation Notes` to the task document.
11. Move the task to `Testing` in `TASKS.md`.
12. In manual mode, stop and hand off to `simplify`.
13. In auto mode, continue through the portable automation chain.

## Reading Rules

Read these by default:

- `AGENTS.md`
- `LEARNINGS.md` if present
- the task document
- files listed in `File Changes`

Avoid broad codebase exploration. If the task document lacks necessary context, use targeted search for the specific symbol, file, route, or command and record the reason in `Implementation Notes`.

## Implementation Notes

Append this section to the task document:

```markdown
## Implementation Notes

### What Changed
- {Summary}

### Files Changed
- `{path}` - {reason}

### Deviations From Plan
- {None, or describe with rationale}

### Verification Run
- `{command}` - PASS | FAIL | SKIPPED ({reason})
```

## Auto Chain

When invoked as `implement auto {ID}`, continue after implementation:

1. `simplify {ID}`
2. `test {ID}`
3. `document {ID}`
4. `ship {ID}`

If the runtime supports background workers or subagents, the adapter may run stages in isolated workers. If it does not, run stages sequentially in the current session. Each stage must write durable details to the task document or its report file.

Stop after three fix-and-retry cycles for the same failing test or quality issue and report the blocker to the user.

## Handoff

Manual mode output:

```text
Implementation complete for task {ID}
Task status: Testing
Next stage: simplify {ID}
```

Auto mode output should summarize the final stage reached and any blocker.
