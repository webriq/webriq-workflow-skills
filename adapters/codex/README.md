# Codex CLI Adapter

This adapter documents Codex-specific usage for the workflow skills. It does not redefine the core workflow in `AGENTS.md` or the skill files.

## Install

```bash
npx skills add eljun/workflow-skills -g -a codex
```

Project install:

```bash
npx skills add eljun/workflow-skills -a codex
```

Install a single workflow stage:

```bash
npx skills add eljun/workflow-skills --skill task -g -a codex
```

## Expected Install Paths

Vercel `npx skills` installs Codex skills to:

| Scope | Path |
|-------|------|
| Project | `.agents/skills/` |
| Global | `~/.agents/skills/` |

## Invocation

Use the stage names directly in natural language or through the Codex skill picker when available:

```text
Use the task skill to plan this feature.
Use implement for task 3.
Use test to verify task 3.
```

If a UI displays slash commands, treat them as display aliases only. The portable workflow contract is the skill stage name.

## Project Instructions

Codex should read `AGENTS.md` as the portable project instruction file. Skill-specific instructions live in the active `SKILL.md`.

## Tier Mapping

Core skills use execution tiers:

| Tier | Codex Mapping Guidance |
|------|------------------------|
| `fast` | Use lower reasoning effort for small, low-risk changes. |
| `balanced` | Use default reasoning for normal implementation work. |
| `deep` | Use higher reasoning effort for architecture, security, migrations, or high-risk refactors. |

If the active Codex runtime does not expose model or reasoning controls through the workflow, keep the tier as advisory context and let Codex configuration control model selection.

## Subagents And Automation

`implement auto` is the portable automation entry point. If Codex subagents are available, they may be used for independent stages or verification workers. If not, run the auto chain sequentially:

```text
implement -> simplify -> test -> document -> ship
```

Do not bypass task review before implementation.
