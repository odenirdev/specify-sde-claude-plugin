# specify-sde

A modular Software Development Engineering toolkit for **GitHub Copilot** and **Claude Code**. It provides a reusable engineering system for real repositories: shared knowledge, repeatable workflows, specialized agents, and derived documentation that stays tied to specs and code.

---

## Why It Exists

Most AI coding assistants produce generic, shallow outputs when asked to do real engineering work. `specify-sde` raises the quality floor with a composable toolkit built around reusable skills, stack-aware knowledge, and runtime-specific adapters.

It is not a rigid framework. It is a reusable base you can adapt, extend, and keep aligned across multiple agent runtimes.

---

## Mental Model

```text
Specs  ──────────────────────────────────────▶  Source of truth
  │                                              (./.specify/specs)
  │
  ├── Knowledge ─────────────────────────────▶  Shared engineering guidance
  │     (languages, frameworks, libs, practices)  reused across runtimes
  │
  ├── Skills ────────────────────────────────▶  Reusable workflows
  │     (define, review, debug, plan, docs-sync)
  │
  ├── Runtime Adapters ──────────────────────▶  GitHub Copilot / Claude Code entrypoints
  │     (.github/, agents/, plugin.json)
  │
  └── Docs ──────────────────────────────────▶  Derived documentation
        (./.specify/docs)                        maintained from specs and code
```

| Layer | Purpose |
|---|---|
| **Specs** | Source of truth. Requirements, definitions, plans, decisions. |
| **Knowledge** | Shared engineering references — practices, patterns, anti-patterns. |
| **Skills** | Reusable workflows that can be loaded by different runtimes. |
| **Runtime Adapters** | Thin Copilot/Claude entrypoints that reference the shared core. |
| **Docs** | Derived documentation. Synced from specs and code. Never invented. |

---

## Directory Structure

```text
specify-sde/
  .github/
    copilot-instructions.md   # Shared GitHub Copilot rules for the workspace
    agents/                   # Copilot agent adapters (*.agent.md)
    skills/                   # Copilot skill adapters (SKILL.md)

  knowledge/
    languages/                # Language guidance (TypeScript, Go, ...)
    frameworks/               # Framework guidance (React, NestJS, LangGraph, ...)
    libs/                     # Library guidance (Prisma, Axios, Vite, ...)
    utilities/                # Cross-cutting guidance (errors, logging, testing)
    practices/                # Engineering practices (hexagonal, docs, tasks)
    platforms/                # Platform/runtime notes (AWS Lambda, ...)

  skills/                     # Shared workflow source of truth
    engineer-discovery/
    engineer-define/
    engineer-define-spec/
    engineer-define-tasks/
    engineer-delivery/
    engineer-review/
    engineer-debug/
    engineer-tradeoff/
    docs-sync/
    knowledge-update/
    knowledge-link/
    agent-link/

  agents/                     # Legacy Claude-oriented agent adapters
    reviewer.md
    backend-architect.md
    debugger.md
    task-planner.md
    docs-maintainer.md
    langgraph-architect.md

  .specify/docs/              # Derived documentation for this toolkit
  plugin.json                 # Legacy plugin metadata / compatibility layer
  CLAUDE.md                   # Bridge guidance for Claude-oriented workflows
```

> Full documentation is maintained in [./.specify/docs/index.md](./.specify/docs/index.md).

---

## Shared Source of Truth

- **Global runtime rules**: [`.github/copilot-instructions.md`](./.github/copilot-instructions.md)
- **Shared engineering knowledge**: [`knowledge/`](./knowledge/)
- **Reusable workflows**: [`skills/`](./skills/)
- **Copilot adapters**: [`.github/agents/`](./.github/agents/) and [`.github/skills/`](./.github/skills/)
- **Claude compatibility layer**: [`agents/`](./agents/) and [`plugin.json`](./plugin.json)

The repository favors **reference over copy**: shared rules and workflow logic live once, while runtime adapters stay thin and discoverable.

---

## Skills Reference

| Skill | Trigger | Output |
|---|---|---|
| `engineer-discovery` | Start a new feature, capture requirements | `prd.md` with problem, goals, requirements, acceptance criteria |
| `engineer-define` | Run full define phase end-to-end | `spec.md` + `tasks.md` in one step |
| `engineer-define-spec` | Generate spec from `prd.md` | `spec.md` with scope, approach, constraints |
| `engineer-define-tasks` | Break spec into executable tasks | `tasks.md` with phased tasks and acceptance criteria |
| `engineer-delivery` | Execute the task plan | Implementation progress + delivery report |
| `engineer-review` | Review code, diff, or module | Structured review with risks and recommendations |
| `engineer-debug` | Investigate a bug or failure | Root-cause hypotheses, evidence, and steps |
| `engineer-tradeoff` | Compare implementation options | Comparison matrix, pros/cons, recommendation |
| `docs-sync` | Update `./.specify/docs` | Sync report: updated, verified, gaps found |

---

## Agents Reference

| Agent | Role | Use When |
|---|---|---|
| `reviewer` | Senior reviewer | Reviewing code or diffs before merge/deploy |
| `backend-architect` | Backend architect | Designing APIs, persistence, or integrations |
| `debugger` | Failure investigator | Diagnosing bugs or production failures |
| `task-planner` | Implementation planner | Converting specs into executable task plans |
| `docs-maintainer` | Documentation engineer | Keeping docs aligned with specs and code |
| `langgraph-architect` | AI orchestration architect | Designing LangGraph / LangChain / Lambda flows |

---

## Usage

### GitHub Copilot in VS Code

1. Open the repository in VS Code with **GitHub Copilot Chat** enabled.
2. Keep the workspace customization files under [`.github/`](./.github/) checked in.
3. Use:
   - `copilot-instructions.md` for global guidance,
   - the agent picker for `reviewer`, `debugger`, `task-planner`, etc.,
   - slash commands such as `/engineer-review` or `/docs-sync`.
4. Prefer the shared `knowledge/` and `skills/` content as the source of truth when extending the system.

### Claude Code compatibility

This repository still supports the existing Claude-oriented adapter flow.

#### 1. Register the local marketplace (once)

```bash
claude plugin marketplace add /Users/odenirgomes/.claude/plugins/marketplaces/local
```

#### 2. Install the plugin

```bash
claude plugin install specify-sde@local
```

#### 3. Verify

```bash
claude plugin list
```

Expected output:

```text
❯ specify-sde@local
  Version: 0.1.0
  Scope: user
  Status: ✔ enabled
```

If you change files under this repository and want to refresh the Claude cache:

```bash
claude plugin uninstall specify-sde@local && claude plugin install specify-sde@local
```

---

## Extending and Maintaining

When adding or changing behavior:

1. Update shared rules in [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) if the rule is global.
2. Update the shared source of truth in [`knowledge/`](./knowledge/) or [`skills/`](./skills/).
3. Keep the Copilot adapters in [`.github/`](./.github/) and the Claude adapters in [`agents/`](./agents/) aligned.
4. Run `docs-sync` to refresh [`.specify/docs/`](./.specify/docs/) when architecture or usage changes.

---

## Core Principles

1. **Specs are the source of truth** — read `./.specify/specs` before acting
2. **Docs are derived** — never invent content from memory or speculation
3. **Knowledge is shared** — reuse `knowledge/` instead of copying guidance around
4. **Skills are reusable** — keep workflows generic and composable
5. **Adapters stay thin** — `.github/` and `agents/` should reference the shared core
6. **Everything is actionable** — vague guidance has no place here
7. **Minimal ceremony** — use what helps, ignore what does not
