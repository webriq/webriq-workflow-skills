---
name: document
description: Use this skill after verification passes and the user approves the result. Updates feature docs, user guides, task retrospectives, LEARNINGS.md, and durable project instructions when the completed task changes reusable workflow knowledge.
---

# document - Documentation Stage

## Invariants

- Run only after verification passed and the user approved the task.
- Base documentation on the task document, test report, and actual changed files.
- Update `LEARNINGS.md` for reusable lessons.
- Update `AGENTS.md` only for stable project-level instructions.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `document`.

## Workflow

1. Read `AGENTS.md`.
2. Resolve the task ID.
3. Read the task document and test report.
4. Review `git diff --name-only` to identify actual changed files.
5. Create or update feature documentation in `docs/features/` when developer-facing documentation is needed.
6. Create or update user guides in `docs/guides/` when user-facing behavior changed.
7. Write a retrospective in `docs/learnings/{ID}-{task-name}.md`.
8. Append reusable lessons to `LEARNINGS.md` without duplicating existing entries.
9. Update `AGENTS.md` only if the task established durable project-level behavior.
10. Move the task to `Approved` in `TASKS.md`.

## Documentation Triggers

| Change | Documentation |
|--------|---------------|
| New user-facing feature | Feature doc and user guide |
| Bug fix with user-visible behavior | Troubleshooting or guide update |
| API or CLI behavior | Developer docs or reference notes |
| New durable project convention | `AGENTS.md` |
| Reusable lesson or gotcha | `LEARNINGS.md` |
| Test setup learning | `LEARNINGS.md` and test report notes |

## Retrospective Template

```markdown
# Task {ID} Retrospective

## Plan Vs Reality
- {What matched or changed}

## Quality Gate Findings
- {What simplify caught}

## Verification Findings
- {What test found}

## Reusable Lessons
- {Actionable lesson}
```

## Handoff

```text
Documentation complete for task {ID}
Task status: Approved
Next stage: ship {ID}
```
