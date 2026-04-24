# Claude Code Adapter

This adapter documents Claude-specific usage for the workflow skills. It does not redefine the core workflow in `AGENTS.md` or the skill files.

## Install

```bash
npx skills add eljun/workflow-skills -g -a claude-code
```

Project install:

```bash
npx skills add eljun/workflow-skills -a claude-code
```

List skills before installing:

```bash
npx skills add eljun/workflow-skills --list
```

## Invocation

Claude Code may expose installed skills as slash commands. Treat those commands as Claude UI conveniences.

Portable stage names remain:

```text
task -> implement -> simplify -> test -> document -> ship -> release
```

Do not use planning automation. The `task` stage stops after creating or updating the task document. Start automation only from an approved task document with `implement auto {task}`.

## Observed Install Paths

With `skills@1.5.1`, temporary verification showed:

| Scope | Path |
|-------|------|
| Project | `.claude/skills/` |
| Global | `~/.claude/skills/` |

## Project Instructions

`AGENTS.md` is the portable source of project instructions.

If a target project retains `CLAUDE.md`, keep it as either:

- a short shim pointing to `AGENTS.md`, or
- a documented mirror whose precedence relative to `AGENTS.md` is clear.

Core workflow operation must not require `CLAUDE.md`.

## Runtime Configuration

Tracked `.claude/` files are Claude runtime configuration. They may contain permissions or local execution preferences, but they do not override core workflow semantics.

The repository currently keeps `.claude/settings.json` as Claude-specific configuration.

## Tier Mapping

Core skills use execution tiers:

| Tier | Claude Mapping Guidance |
|------|-------------------------|
| `fast` | Use a low-cost model or normal reasoning for simple changes. |
| `balanced` | Use the default coding-capable model for most tasks. |
| `deep` | Use the strongest available reasoning/model configuration for architecture, security, migrations, and high-risk refactors. |

Concrete model names are Claude runtime choices and may change over time. Do not hardcode Claude model names in core skills.

## Tool Notes

Claude-specific MCP setup, browser tools, and permission examples belong here or in the target project. Core `test` guidance is capability-based and does not require one specific browser automation tool.
