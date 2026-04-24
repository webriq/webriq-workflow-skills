# Gemini CLI Adapter

This adapter documents Gemini CLI install compatibility for the workflow skills. It does not redefine the core workflow in `AGENTS.md` or the skill files.

## Install

```bash
npx skills add eljun/workflow-skills -g -a gemini-cli
```

Project install:

```bash
npx skills add eljun/workflow-skills -a gemini-cli
```

List available skills:

```bash
npx skills add eljun/workflow-skills --list
```

## Expected Install Paths

Vercel `npx skills` installs Gemini CLI skills to:

| Scope | Path |
|-------|------|
| Project | `.agents/skills/` |
| Global | `~/.agents/skills/` |

## Support Level

This migration verifies install compatibility for Gemini CLI through `npx skills`.

Runtime-specific invocation, permissions, and tier mapping should be validated in a Gemini CLI environment before claiming full runtime parity.

## Tier Mapping

Core skills use execution tiers:

| Tier | Gemini Mapping Guidance |
|------|-------------------------|
| `fast` | Advisory for small, low-risk tasks. |
| `balanced` | Advisory for normal implementation work. |
| `deep` | Advisory for high-risk or architecture-heavy work. |

If Gemini CLI does not expose explicit model or reasoning routing through the workflow, keep the tier as task context only.
