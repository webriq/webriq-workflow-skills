---
name: simplify
description: Use this skill after implementation and before testing to run a quality gate. Reviews changed files against the task document, checks maintainability standards, classifies plan deviations, and decides whether the work can proceed to verification.
---

# simplify - Quality Gate Stage

## Invariants

- Review only the task scope and changed files.
- Do not add new product scope.
- Block major deviations before testing.
- Write durable findings to the task document.

Runtime adapters may expose this stage as a slash command, menu action, or natural-language skill invocation. The portable stage name is `simplify`.

## Workflow

1. Read `AGENTS.md`.
2. Resolve the task ID and read `docs/task/{ID}-{task-name}.md`.
3. Identify changed files from `Implementation Notes` and `git diff --name-only`.
4. Read changed files only.
5. Check coding standards and maintainability.
6. Compare implementation against requirements, file changes, out-of-scope boundaries, and acceptance criteria.
7. Classify deviations.
8. Append `Quality Gate Notes` to the task document.
9. Return PASS or FAIL.

## Standards Checklist

Apply where relevant:

- no unused code, dead code, or commented-out implementation
- no broad `any` or untyped escape hatches when the project has typed alternatives
- no deep nesting where guard clauses would be clearer
- functions and files have clear responsibility
- names describe behavior accurately
- repeated logic is extracted when repetition creates maintenance risk
- errors are handled intentionally
- no secrets, credentials, or debug logging in production paths
- project conventions from `AGENTS.md`, `LEARNINGS.md`, and the task doc are followed

## Deviation Classification

| Level | Meaning | Outcome |
|-------|---------|---------|
| Minor | Small implementation difference that still satisfies scope. | Document and proceed. |
| Medium | Meaningful difference that should be visible to the user. | Document and proceed only if risk is acceptable. |
| Major | Violates requirements, expands scope, or changes architecture without approval. | FAIL and stop. |

## Output In Task Document

```markdown
## Quality Gate Notes

### Result
PASS | FAIL

### Standards Review
- {Finding or "No blocking issues"}

### Deviations
- {Deviation level and rationale}

### Required Fixes
- {Only if FAIL}
```

## Handoff

PASS:

```text
Quality gate passed for task {ID}
Next stage: test {ID}
```

FAIL:

```text
Quality gate failed for task {ID}
Return to implementation with the required fixes listed in the task document.
```
