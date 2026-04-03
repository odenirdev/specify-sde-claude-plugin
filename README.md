# specify-sde

A modular Software Development Engineering toolkit for Claude Code. Designed to help Claude act like a strong software engineer working inside real repositories — reading specs, producing concrete designs, reviewing code rigorously, and keeping documentation aligned with reality.

---

## Why It Exists

Most AI coding assistants produce generic, shallow outputs when asked to do real engineering work. `specify-sde` gives Claude Code a structured engineering system — a set of reusable capabilities, domain knowledge, and specialized roles — that raises the quality floor for every engineering task.

It is not a rigid framework. It is a composable toolkit. Use the pieces that are useful. Ignore the rest.

---

## Mental Model

```
Specs  ──────────────────────────────────────▶  Source of truth
  │                                              (./.specify/specs)
  │
  ├── Skills ──────────────────────────────────▶  Reusable capabilities
  │     │                                         (define, review, debug, plan)
  │     │
  │     └── Agents ──────────────────────────▶   Specialized roles
  │           (reviewer, architect, debugger)     that compose skills
  │
  ├── Knowledge ───────────────────────────────▶  Stack-aware guidance
  │     (TypeScript, Go, NestJS, Prisma, ...)     for engineering decisions
  │
  └── Docs ─────────────────────────────────────▶  Derived documentation
        (./.specify/docs)                           maintained by docs-sync
```

| Layer | Purpose |
|---|---|
| **Specs** | Source of truth. Requirements, definitions, plans, decisions. |
| **Docs** | Derived documentation. Synced from specs and code. Never invented. |
| **Skills** | Reusable engineering capabilities — loaded into any conversation. |
| **Agents** | Specialized roles that compose skills for specific job types. |
| **Knowledge** | Stack-aware engineering references — practices, patterns, anti-patterns. |
| **Templates** | Starting structures for `./.specify/docs` and project conventions. |

---

## Directory Structure

```
specify-sde/
  .claude-plugin/
    plugin.json               # Plugin manifest

  knowledge/
    languages/
      typescript.md           # TypeScript best practices + review criteria
      go.md                   # Go idioms, error handling, concurrency
    frameworks/
      react.md                # React patterns, state management, data fetching
      nestjs.md               # NestJS modules, DI, DTOs, exceptions
    libs/
      prisma.md               # Prisma client, migrations, query patterns
      axios.md                # HTTP client setup, interceptors, error handling
    utilities/
      error-handling.md       # Error wrapping, domain errors, boundary mapping
      logging.md              # Structured logging, levels, what not to log
      testing.md              # Test layers, AAA, fakes vs mocks, coverage
    practices/
      api-design.md           # REST conventions, status codes, versioning
      task-breakdown.md       # Phased tasks, acceptance criteria, dependencies
      hexagonal-architecture.md  # Ports & adapters, dependency flow, examples
      documentation-derivation.md  # What to document, derivation rules, ADRs

  skills/
    engineer-discovery/       # Requirements interview + prd.md generation
    engineer-define/          # Define phase orchestrator (spec + tasks)
    engineer-define-spec/     # Spec generation from prd.md
    engineer-define-tasks/    # Task breakdown from spec.md
    engineer-delivery/        # Execute task plan via subagents
    engineer-review/          # Code and diff review
    engineer-debug/           # Root cause analysis and debugging strategy
    engineer-tradeoff/        # Option comparison and recommendation
    docs-sync/                # Sync ./.specify/docs from specs and code

  agents/
    reviewer.md               # Senior reviewer — production risk, architecture
    backend-architect.md      # Backend design — APIs, persistence, integrations
    debugger.md               # Failure investigator — root cause analysis
    task-planner.md           # Spec-to-task converter — phased delivery plans
    docs-maintainer.md        # Documentation engineer — accuracy and currency

  templates/
    specify-docs-index-template.md      # Template for ./.specify/docs/index.md
    specify-architecture-template.md    # Template for architecture.md
    specify-integrations-template.md    # Template for integrations.md
    specify-operations-template.md      # Template for operations.md
    project-conventions-template.md     # Template for project conventions
```

> Full documentation is maintained in [./.specify/docs/index.md](./.specify/docs/index.md) — architecture, stack, operations, and decision records.

---

## Skills Reference

| Skill | Trigger | Output |
|---|---|---|
| `engineer-discovery` | Start a new feature, capture requirements | `prd.md` with problem, goals, requirements, acceptance criteria |
| `engineer-define` | Run full define phase end-to-end | `spec.md` + `tasks.md` in one step |
| `engineer-define-spec` | Generate spec from prd.md | `spec.md` with scope, approach, constraints |
| `engineer-define-tasks` | Break spec into executable tasks | `tasks.md` with phased tasks and acceptance criteria |
| `engineer-delivery` | Execute the task plan | Implementation via subagents + `delivery.md` report |
| `engineer-review` | Review code, diff, or module | Structured review: critical issues, improvements, strengths |
| `engineer-debug` | Investigate a bug or failure | Root cause hypotheses, debugging steps, evidence to collect |
| `engineer-tradeoff` | Compare implementation options | Comparison matrix, pros/cons, recommendation |
| `docs-sync` | Update ./.specify/docs | Sync report: updated, verified, gaps found |

---

## Agents Reference

| Agent | Role | Use When |
|---|---|---|
| `specify-sde:reviewer` | Senior reviewer | Reviewing code or diffs before merge/deploy |
| `specify-sde:backend-architect` | Backend architect | Designing APIs, persistence, or integrations |
| `specify-sde:debugger` | Failure investigator | Diagnosing bugs or production failures |
| `specify-sde:task-planner` | Implementation planner | Converting specs into executable task plans |
| `specify-sde:docs-maintainer` | Documentation engineer | Keeping docs aligned with specs and code |

---

## Installation

This plugin lives in the `local` marketplace. Make sure the marketplace is registered before installing.

### 1. Register the local marketplace (once)

```bash
claude plugin marketplace add /Users/odenirgomes/.claude/plugins/marketplaces/local
```

> Skip this step if the `local` marketplace is already listed in `known_marketplaces.json`.

### 2. Install the plugin

```bash
claude plugin install specify-sde@local
```

### 3. Verify

```bash
claude plugin list
```

Expected output:
```
❯ specify-sde@local
  Version: 0.1.0
  Scope: user
  Status: ✔ enabled
```

Restart Claude Code after installation for skills and agents to take effect.

### Update after source changes

If you modify files inside this directory, reinstall to sync the cache:

```bash
claude plugin uninstall specify-sde@local && claude plugin install specify-sde@local
```

---

## Extending

Authoring rules for skills, agents, knowledge files, and templates are documented in [CLAUDE.md](./CLAUDE.md).

After adding or modifying content, run `docs-sync` to update [./.specify/docs/](./.specify/docs/).

---

## Core Principles

1. **Specs are the source of truth** — read `./.specify/specs` before acting
2. **Docs are derived** — never invented from memory or speculation
3. **Skills are reusable** — compose them for any task
4. **Agents are roles** — specialized for specific job types
5. **Knowledge is practical** — real patterns, real anti-patterns, real trade-offs
6. **Everything is actionable** — vague guidance has no place here
7. **Minimal ceremony** — use what helps, ignore what doesn't
