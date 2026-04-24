---
name: task
description: Use this skill when the user wants to plan a new feature, bug fix, enhancement, documentation update, or chore before implementation. Creates or updates a reviewed task document in docs/task/ and updates TASKS.md, then stops for human approval.
---

# task - Planning Stage

## Invariants

- Create or update the task document only.
- Do not implement code.
- Do not start automation from planning.
- Stop after updating `TASKS.md` and tell the user the task is ready for review.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `task`.

## Workflow

1. Read `AGENTS.md`.
2. Read `LEARNINGS.md` if it exists.
3. Clarify the request only where missing information changes scope, risk, or acceptance criteria.
4. Perform targeted codebase research for files directly relevant to the task.
5. Choose a recommended execution tier.
6. Create or update `docs/task/{ID}-{task-name}.md` using `docs/templates/task-document.md`.
7. Add or update the task in `TASKS.md` under `Planned`.
8. Stop for user review and approval.

## Execution Tier Guidance

Use `Recommended Tier` in the task document:

| Tier | Use When |
|------|----------|
| `fast` | Small, low-risk changes that follow existing patterns. |
| `balanced` | Normal feature work, moderate refactors, or changes across several files. |
| `deep` | Architecture, security, data model, migration, high-risk refactor, or ambiguous requirements. |

The tier is guidance for downstream agents. Runtime adapters define how, or whether, tiers map to models or reasoning settings.

## Task Document Requirements

The task document must include:

- clear overview and requirements
- explicit out-of-scope or must-not-change boundaries
- proposed file changes
- focused code context for files the implementation stage should read
- implementation steps
- acceptance criteria
- verification commands or approach
- compatibility touchpoints for packaging, docs, adapters, or install surface when relevant

Embed concise code excerpts or notes in `Code Context`; do not paste entire unrelated files.

## TASKS.md Update

If `TASKS.md` does not exist, create it from `docs/templates/tasks-tracking.md`.

Add the task to `Planned`:

```markdown
| {ID} | {Task} | {Priority} | {Type} | docs/task/{ID}-{task-name}.md | {Date} |
```

## Handoff

End with:

```text
Task planned: docs/task/{ID}-{task-name}.md
Status: awaiting user review
Next stage after approval: implement {ID}
```
