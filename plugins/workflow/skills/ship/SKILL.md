---
name: ship
description: Use this skill after documentation is complete and the task is approved. Runs pre-ship checks, prepares a branch or pull request, updates TASKS.md, and leaves release timing to the release stage.
---

# ship - Pull Request Stage

## Invariants

- Ship only approved and documented tasks.
- Run available pre-ship checks before creating a pull request.
- Do not auto-merge.
- Keep released-version work for the `release` stage.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `ship`.

## Workflow

1. Read `AGENTS.md`.
2. Resolve the task ID and read the task document.
3. Verify the task is in `Approved` in `TASKS.md`.
4. Confirm feature docs and test report exist where applicable.
5. Run pre-ship checks documented by the project or task.
6. Review changed files for unintended edits and secrets.
7. Create or reuse an appropriate branch.
8. Create a pull request with links to the task document and test report.
9. Move the task to `Ready To Ship` in `TASKS.md`.

## Pre-Ship Checks

Run available project checks, such as:

```bash
pnpm build
pnpm typecheck
pnpm lint
pnpm test
```

Use the commands that exist in the target project. If a command is unavailable, record that it was skipped and why.

## Pull Request Body

```markdown
## Summary

- {Change summary}

## Verification

- {Command or report}

## Workflow References

- Task: docs/task/{ID}-{task-name}.md
- Test report: docs/testing/{ID}-{task-name}.md
- Feature docs: {path or N/A}
```

## Handoff

```text
Ship preparation complete for task {ID}
Task status: Ready To Ship
PR: {url}
Next stage after merge: release
```
