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

| Layer | Responsibility | Canonical source | Thin / derived surfaces |
|---|---|---|---|
| **Engineering Context** | Architecture, operations, ADRs, and navigation for this toolkit | `./.specify/docs/` | `README.md`, `CLAUDE.md` |
| **Governance** | Shared maintenance rules and authoring expectations | `.github/copilot-instructions.md` | Referenced from docs and runtime bridges |
| **Domain** | Stable shared references and engineering conventions | `references/` | Consumed by skills and agents |
| **Use Cases** | Reusable workflows for discovery, definition, review, debug, and docs sync | `skills/*/SKILL.md` | `.github/skills/*/SKILL.md` |
| **Agent Contracts** | Full role-specific guidance for named agents | `agents/*.md` | `.github/agents/*.agent.md` |
| **Runtime Adapters** | Metadata and entrypoints for GitHub Copilot and Claude Code | `.github/`, `plugin.json` | N/A |

This keeps the dependency direction aligned with Clean / Hexagonal Architecture: **shared core first, runtime adapters last**. Edit the canonical source first; update the thin surfaces only to keep discovery metadata and links aligned.

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

## Canonical Ownership and Duplication Map

| Concern | Canonical source | Thin or derived files | Maintenance rule |
|---|---|---|---|
| Architecture and navigation | `./.specify/docs/index.md` and sibling docs | `README.md`, `CLAUDE.md` | Keep entrypoints short and link back to docs |
| Shared rules | `.github/copilot-instructions.md` | Referenced by agents and docs | Do not restate the same rule in every adapter |
| Reusable workflows | `skills/*/SKILL.md` | `.github/skills/*/SKILL.md` | Copilot skill files should remain wrappers that point to the source workflow |
| Agent role guidance | `agents/*.md` | `.github/agents/*.agent.md` | Copilot agent files should keep only runtime metadata, mission summary, and canonical links |
| Shared technical guidance | `references/**/*.md` | Mentioned from skills or agents | Prefer links over copied prose |

### Adapter Contract

`.github/skills/*/SKILL.md` and `.github/agents/*.agent.md` are intentionally thin. They may contain:
- runtime frontmatter and tool metadata;
- a short mission / when-to-use summary;
- links to the canonical workflow, agent file, and shared references.

They should **not** become second sources of truth for long examples, detailed constraints, or copied architecture prose. When a wrapper starts growing, move the shared content back to `skills/`, `agents/`, `references/`, or `./.specify/docs/`.

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
