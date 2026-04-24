# Workflow Skills

Agent Skills-compatible development workflow for planning, implementing, verifying, documenting, shipping, and releasing software changes.

The workflow is memory-first: each stage reads and writes durable project files instead of relying on chat history.

## Stages

| Stage | Purpose | Main Output |
|-------|---------|-------------|
| `task` | Plan work and create a reviewed task document | `docs/task/*.md` |
| `implement` | Apply an approved task document | code/content changes |
| `simplify` | Quality gate and deviation review | task doc notes |
| `test` | Capability-based verification | `docs/testing/*.md` |
| `document` | Feature docs, guides, and learnings | `docs/features/`, `docs/guides/`, `LEARNINGS.md` |
| `ship` | Pre-ship checks and pull request | PR and `TASKS.md` update |
| `release` | Versioned changelog and release | tag/release notes/changelog |

Runtime adapters may expose these stages as slash commands, menus, or natural-language skill invocations. The portable contract is the stage name.

## Install

This repository is distributed through Vercel `npx skills`.

List available skills:

```bash
npx skills add eljun/workflow-skills --list
```

Install all workflow skills globally:

```bash
npx skills add eljun/workflow-skills -g
```

Install to a specific agent:

```bash
npx skills add eljun/workflow-skills -g -a codex
npx skills add eljun/workflow-skills -g -a claude-code
npx skills add eljun/workflow-skills -g -a gemini-cli
```

Install a single skill:

```bash
npx skills add eljun/workflow-skills --skill task -g -a codex
```

Install all skills to all detected agents:

```bash
npx skills add eljun/workflow-skills --all
```

Project install is the default when `-g` is omitted. Interactive installs can use symlinks or copies; symlinks are recommended for local development because they keep one canonical source.

Agent selection with `-a` chooses the install target. It does not choose a model. Model or reasoning choices are runtime-specific and documented in `adapters/`.

## Canonical Layout

The canonical skill source for this repository is:

```text
plugins/workflow/skills/{skill-name}/SKILL.md
```

The root `.claude-plugin/marketplace.json` is retained as compatibility metadata for `npx skills` manifest discovery and Claude plugin compatibility. It is not the portable source of workflow semantics.

## Project Setup

In the target project, create the workflow memory folders:

```bash
mkdir -p docs/task docs/testing docs/features docs/guides docs/changelogs docs/learnings
```

Recommended tracked files:

| Path | Purpose |
|------|---------|
| `AGENTS.md` | Portable project instructions |
| `TASKS.md` | Workflow tracker |
| `LEARNINGS.md` | Mutable project lessons and gotchas |
| `docs/task/` | Task documents |
| `docs/testing/` | Test reports |

## Workflow

Manual flow:

```text
task -> implement -> simplify -> test -> document -> ship -> release
```

Planning always stops for review. The `task` stage creates or updates the task document and `TASKS.md`, then waits for user approval.

Automation starts only from implementation:

```text
implement auto {task}
```

This assumes the task document already exists and has been reviewed. The auto chain is:

```text
implement -> simplify -> test -> document -> ship
```

The workflow does not auto-merge pull requests or auto-release changes.

## Execution Tiers

Core workflow language uses tiers:

| Tier | Meaning |
|------|---------|
| `fast` | Small, low-risk work following existing patterns |
| `balanced` | Normal feature work or moderate refactors |
| `deep` | Architecture, security, data model, migrations, or high-risk work |

Adapters map these tiers to runtime-specific model or reasoning settings when the runtime supports it. If a runtime has no explicit model control, the tier remains guidance.

## Verification

The `test` stage is capability-based. It can use:

- unit tests
- integration tests
- browser or end-to-end automation
- build, typecheck, lint, or CI checks
- manual inspection when automation is unavailable

Tool-specific setup belongs in adapter docs or the target project, not the core workflow contract.

## Adapters

Runtime notes live in:

| Adapter | Path |
|---------|------|
| Claude Code | `adapters/claude/README.md` |
| Codex CLI | `adapters/codex/README.md` |
| Gemini CLI | `adapters/gemini/README.md` |

Adapters explain install paths, invocation style, permissions, model or reasoning tier mappings, and tool setup. They do not redefine the workflow stages.

## Companion Skills

Install companion skills separately when relevant to the target project:

```bash
npx skills add vercel-labs/agent-skills
npx skills add supabase/agent-skills
```

Companion skills are optional. Use them when their descriptions match the task and the agent has them installed.
