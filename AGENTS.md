# Workflow Skills Project Instructions

## Purpose

This repository packages a memory-first development workflow as Agent Skills installable with Vercel `npx skills`.

The workflow stages are:

1. `task`
2. `implement`
3. `simplify`
4. `test`
5. `document`
6. `ship`
7. `release`

The core workflow is agent-neutral. Runtime-specific behavior for Claude Code, Codex CLI, Gemini CLI, or other agents belongs in `adapters/`.

## Instruction Hierarchy

Follow this precedence order:

1. `AGENTS.md`
2. The active workflow `SKILL.md`
3. Runtime adapter docs in `adapters/`
4. Explanatory docs such as `README.md`

`.claude/` configuration and `.claude-plugin/` metadata are compatibility and install surfaces. They do not define portable workflow semantics.

## Workflow Philosophy

- Use task documents as shared working memory between stages.
- Keep each stage focused on one responsibility.
- Prefer concise, high-signal instructions over large orchestration frameworks.
- Preserve human review before implementation.
- Separate skill installation from model or reasoning selection.
- Use execution tiers in core workflow language: `fast`, `balanced`, and `deep`.

## Task Document Rules

Task documents live in `docs/task/`.

The `task` stage creates or updates a task document, updates `TASKS.md`, and then stops. It must not start implementation.

Implementation can begin only after task review is complete and the task document has been approved by the user. `implement auto` is the only supported automation entry point, and it assumes an approved task document already exists.

Test reports live in `docs/testing/`.

## Project Memory

`LEARNINGS.md` is the mutable project knowledge base. If it exists, workflow stages should read it before making decisions that could repeat known mistakes.

If `LEARNINGS.md` does not exist, do not pretend it does. Create it only when a workflow stage captures a reusable project lesson.

Use `AGENTS.md` for stable project-level instructions. Use `LEARNINGS.md` for accumulated lessons, repeated mistakes, project patterns, and durable implementation notes.

## Adapter Boundaries

Adapters may document:

- install commands for a specific agent
- runtime invocation style
- permission examples
- model or reasoning-tier mappings
- tool availability and setup notes

Adapters must not redefine the core workflow stages, task-review requirement, task document paths, or execution-tier vocabulary.

## Stop And Ask

Stop and ask the user instead of improvising when:

- a task document is missing or has not been approved for implementation
- the requested change conflicts with `AGENTS.md` or the active `SKILL.md`
- a required verification step cannot run and no safe fallback exists
- install discovery through `npx skills` breaks
- a change would require maintaining duplicate unsynchronized skill files
- a runtime-specific command is needed but not documented in an adapter
- the requested operation is destructive or would overwrite unrelated user work
