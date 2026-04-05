---
updated_at: 2026-04-05T00:00:00Z
---

# Architecture — specify-sde

> High-level toolkit structure for shared engineering content and runtime adapters.

---

## System Overview

`specify-sde` is a Markdown-first customization toolkit. There is no runtime process, build system, or compiled output. The repository now follows a **shared core + runtime adapters** model: `references/` holds reusable engineering guidance, `skills/` holds workflows, and `.github/` plus `agents/` expose that shared core to GitHub Copilot and Claude Code. When workflows write artifacts, they operate on the consuming project's `./.specify/` directory.

---

## Layered View

| Layer | Responsibility | Repository paths |
|---|---|---|
| **Domain** | Stable shared references and engineering conventions | `references/` |
| **Use Cases** | Reusable workflows for discovery, definition, review, debug, and docs sync | `skills/*/SKILL.md` |
| **Adapters** | Runtime-specific entrypoints for GitHub Copilot and Claude Code | `.github/`, `agents/`, `plugin.json` |
| **Derived Docs** | Architecture and operations notes generated from repository evidence | `./.specify/docs/` |

This keeps the dependency direction aligned with Clean / Hexagonal Architecture: **shared core first, runtime adapters last**.

---

## Component Map

```text
specify-sde/
  references/<category>/      ─── Shared engineering references (domain)
  │  Languages, frameworks, libraries, utilities, and practices.
  │  Runtime-agnostic and reused across every adapter.

  skills/<name>/SKILL.md     ─── Reusable workflows (use cases)
  │  Discovery, define, review, debug, docs-sync, and maintenance flows.
  │  These are the behavior source of truth.

  .github/                   ─── GitHub Copilot adapters
  │  copilot-instructions.md ─── Shared workspace rules for Copilot
  │  agents/*.agent.md       ─── Thin role adapters with minimal tool sets
  │  skills/*/SKILL.md       ─── Thin skill adapters that point back to `skills/`

  agents/<name>.md           ─── Claude compatibility adapters
  │  Existing role definitions kept for transition and backward compatibility.

  plugin.json                ─── Legacy metadata for Claude-oriented installation

  .specify/docs/             ─── Derived docs for this repository
  │  stack.md, architecture.md, operations.md, decisions/
```

---

## Components

### References (`references/**/*.md`)
- **Responsibility**: Provide stack-aware, actionable engineering guidance and shared principles.
- **Interfaces**: Referenced by skills and adapters rather than copied into them.
- **Dependencies**: None.
- **Location**: [references/](../../references/)

### Skills (`skills/*/SKILL.md`)
- **Responsibility**: Define reusable workflows with explicit inputs, phases, outputs, and quality bars.
- **Interfaces**: Consumed through Copilot wrappers in `.github/skills/` and through Claude-compatible flows.
- **Dependencies**: May reference shared references and supporting references.
- **Location**: [skills/](../../skills/)

### GitHub Copilot Adapters (`.github/`)
- **Responsibility**: Provide discoverable workspace instructions, custom agents, and slash-invocable skills for Copilot.
- **Interfaces**: `copilot-instructions.md`, `.github/agents/*.agent.md`, `.github/skills/*/SKILL.md`.
- **Dependencies**: Must reference the shared `references/` and `skills/` core instead of duplicating it.
- **Location**: [../../.github/](../../.github/)

### Claude Compatibility Layer (`agents/*.md`, `plugin.json`)
- **Responsibility**: Preserve the existing adapter surface for Claude-oriented workflows during the transition.
- **Interfaces**: `agents/*.md` and `plugin.json`.
- **Dependencies**: Same shared core as Copilot.
- **Location**: [agents/](../../agents/), [plugin.json](../../plugin.json)

### Docs Sync (`skills/docs-sync/`)
- **Responsibility**: Detect project structure, activate relevant references, and keep `./.specify/docs/`, root `CLAUDE.md`, and the managed `README.md` block aligned with real repository state.
- **Interfaces**: `init`, `index`, `architecture`, `integrations`, `claude`, `readme`, `all` scopes.
- **Dependencies**: Uses stack maps, agent maps, and templates from `skills/docs-sync/references/`.
- **Location**: [skills/docs-sync/](../../skills/docs-sync/)

---

## Key Relationships

| From | To | Contract |
|---|---|---|
| Copilot agent | Shared skill | Agent wrapper points to the reusable workflow in `skills/` |
| Copilot skill | Shared references | Skill adapter links to the relevant reference files instead of embedding them |
| Claude adapter | Shared core | Legacy adapters remain aligned with the same `references/` and `skills/` source |
| `docs-sync` | `./.specify/docs/` | Updates derived docs from real code/spec evidence only |
| `docs-sync` | `README.md` / `CLAUDE.md` | Keeps the human and agent entrypoints aligned without creating a parallel source of truth |
| Any workflow | `./.specify/specs/` | Writes artifacts with `updated_at` frontmatter in the consuming project |

---

## Authoring Constraints

- Shared runtime rules belong in [`.github/copilot-instructions.md`](../../.github/copilot-instructions.md).
- Shared workflow behavior belongs in `skills/*/SKILL.md`; Copilot wrappers stay intentionally thin.
- Copilot skills must keep `name` aligned with the folder name and use keyword-rich `description` fields.
- Copilot agents must stay single-purpose with the minimum tool set required for the role.
- Reference files should remain the primary place for reusable engineering guidance.
- Derived docs must stay evidence-based and must not introduce speculation.

---

## Architectural Decisions

| ADR | Decision | Status |
|---|---|---|
| [ADR-001](./decisions/ADR-001-dual-runtime-customization.md) | Keep the shared core runtime-agnostic and expose thin adapters for GitHub Copilot and Claude Code | Accepted |
