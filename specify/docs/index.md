---
updated_at: 2026-04-03T00:00:00Z
---

# specify-sde

A modular Software Development Engineering toolkit for Claude Code. Gives Claude a structured engineering system — reusable skills, domain knowledge, and specialized agent roles — to raise the quality floor for every engineering task in any project that installs this plugin.

---

## Stack & Architecture

| Layer | Technology | Notes |
|---|---|---|
| Runtime | None | Pure Markdown plugin, no build step |
| Format | Claude Code plugin system | Declared via `plugin.json` |
| Content | Markdown | Skills, agents, knowledge, templates |

**Architecture pattern**: Composable skill system — skills loaded on trigger, agents compose skills, knowledge informs both.

See [architecture.md](./architecture.md) for full plugin structure and component relationships.

---

## Main Features

- **engineer-discovery**: Requirements interview + `prd.md` generation
- **engineer-define**: Full define phase orchestrator (spec + tasks in one step)
- **engineer-define-spec**: Technical spec from `prd.md`
- **engineer-define-tasks**: Phased task breakdown from spec
- **engineer-delivery**: Execute task plan via specialized subagents
- **engineer-review**: Structured code and diff review
- **engineer-debug**: Root cause analysis and debugging strategy
- **engineer-tradeoff**: Option comparison with recommendation
- **docs-sync**: Synchronize `./specify/docs` from specs and code

---

## Domain

Key concepts in this plugin's model:

- **Spec**: Source of truth for a feature — problem, approach, constraints, tasks. Lives in `./specify/specs/<slug>/`.
- **Skill**: A reusable Claude capability loaded from `skills/<name>/SKILL.md` on trigger. Produces a defined output.
- **Agent**: A specialized role that composes one or more skills. Declared in `agents/<name>.md`.
- **Knowledge**: Stack-aware engineering reference for a technology or practice. Lives in `knowledge/`.
- **Docs**: Derived documentation in `./specify/docs/`. Never invented — always derived from specs or code.

---

## Engineering Agents

Agents configured for this project's stack:

| Agent | Role |
|---|---|
| `specify-sde:debugger` | Root cause analysis and failure investigation |
| `specify-sde:task-planner` | Spec-to-task breakdown and delivery planning |
| `specify-sde:docs-maintainer` | Documentation accuracy and currency |
| `specify-sde:reviewer` | Skill/agent authoring review and quality assessment |

---

## Derived Documentation

| Document | Description |
|---|---|
| [stack.md](./stack.md) | Detected stack, active agents, active knowledge |
| [architecture.md](./architecture.md) | Plugin structure, component map, skill/agent relationships |
| [operations.md](./operations.md) | Install, update, and maintenance procedures |
| [decisions/](./decisions/) | Architectural Decision Records (ADRs) |

---

## Hypotheses & Pending Items

> Unresolved questions and unvalidated assumptions. Remove items when confirmed or refuted.

- [ ] `integrations.md` skipped — no external service integrations detected in current codebase.
