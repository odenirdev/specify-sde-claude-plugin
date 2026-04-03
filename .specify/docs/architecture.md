---
updated_at: 2026-04-03T00:00:00Z
---

# Architecture — specify-sde

> High-level plugin structure. Updated when significant structural changes are made.

---

## System Overview

`specify-sde` is a Claude Code plugin composed entirely of Markdown files. There is no runtime, no build system, and no compiled output. Claude loads skills on trigger, agents compose skills into specialized roles, and knowledge files inform engineering decisions for specific stacks. The plugin operates on the consuming project's `./.specify/` directory as its working area.

---

## Component Map

```
specify-sde/
  plugin.json              ─── Plugin manifest (name, version, description)
  
  skills/<name>/SKILL.md   ─── Reusable capabilities
  │  Loaded by Claude when trigger phrases are matched.
  │  Each skill defines: objective, phases, output location, quality bar.
  │  Output → consuming project's ./.specify/specs/<slug>/<type>.md
  
  agents/<name>.md         ─── Specialized roles
  │  Each agent composes one or more skills.
  │  Triggered by Claude Agent SDK subagent dispatch.
  
  knowledge/<category>/    ─── Stack-aware engineering references
  │  Loaded by skills and agents based on detected stack.
  │  Informs interpretation and documentation — never speculative.
  
  skills/docs-sync/
    references/            ─── Docs sync operating data
      stack-signals.md     ─── Files to inspect for stack detection
      knowledge-map.md     ─── Stack → knowledge file mapping
      agent-map.md         ─── Stack → agent mapping
      templates.md         ─── Template → target file mapping
      template-*.md        ─── Templates for each docs/ file
```

---

## Components

### Skills (`skills/*/SKILL.md`)
- **Responsibility**: Define a reusable engineering capability with explicit inputs, phases, and output format.
- **Interfaces**: Triggered by Claude via natural language; writes output to `./.specify/specs/<slug>/` in consuming project.
- **Dependencies**: May reference knowledge files and invoke subagents.
- **Location**: [skills/](../../skills/)

### Agents (`agents/*.md`)
- **Responsibility**: Define a specialized Claude role that composes skills for a specific job type (review, debug, architect, plan).
- **Interfaces**: Triggered via Claude Agent SDK subagent dispatch or directly by the user.
- **Dependencies**: Compose one or more skills; reference knowledge files.
- **Location**: [agents/](../../agents/)

### Knowledge (`knowledge/**/*.md`)
- **Responsibility**: Provide stack-aware, actionable engineering guidance (best practices, anti-patterns, review criteria, trade-offs).
- **Interfaces**: Referenced by skills and agents under "Knowledge to Consult" / "Knowledge to Prioritize".
- **Dependencies**: None.
- **Location**: [knowledge/](../../knowledge/)

### Docs Sync (`skills/docs-sync/`)
- **Responsibility**: Detect consuming project's stack, activate matching knowledge and agents, and synchronize `./.specify/docs/` with current specs and code.
- **Interfaces**: Accepts scope arguments: `init`, `index`, `architecture`, `integrations`, `all`.
- **Dependencies**: References stack-signals, knowledge-map, agent-map, and templates from its `references/` directory.
- **Location**: [skills/docs-sync/](../../skills/docs-sync/)

---

## Key Relationships

| From | To | Contract |
|---|---|---|
| Agent | Skill | Agent declares which skills it composes in "Skills Used" section |
| Skill | Knowledge | Skill declares files to load in "Knowledge to Consult" section |
| docs-sync | templates | Reads templates from `references/`, fills with codebase data, writes to `./.specify/docs/` |
| Any skill | `./.specify/specs/` | Skills write output with `updated_at` frontmatter; overwrite if exists |

---

## Authoring Constraints

- Skills must include YAML frontmatter with `name`, `description`, `argument-hint`, `allowed-tools`.
- Agents must include YAML frontmatter with `name`, `description`, `tools`, `model`, `color`.
- Knowledge files must cover: Best Practices, Anti-Patterns, Review Criteria, Trade-offs, Implementation Notes.
- All spec artifacts written by skills must include `updated_at: YYYY-MM-DDTHH:MM:SSZ` frontmatter.
- Templates in `docs-sync/references/` must be filled with real data — placeholders must not appear in output.

---

## Architectural Decisions

| ADR | Decision | Status |
|---|---|---|
| — | No ADRs recorded yet | — |
