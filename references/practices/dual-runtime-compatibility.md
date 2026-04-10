# Dual-Runtime Compatibility — Engineering Reference

## Core Principle

Shared core first, runtime adapters last. Logic defined once in `references/`, `skills/`, and `agents/` — exposed twice through thin wrappers for GitHub Copilot and Claude Code. An adapter that grows beyond a wrapper has become a second source of truth, and two sources of truth always drift.

## Runtime Layout

```text
specify-sde/
  references/<category>/        ← shared engineering guidance (runtime-agnostic)
  skills/<name>/SKILL.md        ← canonical workflow definition
  agents/<name>.md              ← canonical agent persona and instructions

  .github/                      ← GitHub Copilot adapter surface
    copilot-instructions.md     ← shared workspace rules
    agents/<name>.agent.md      ← thin Copilot wrapper → links to agents/<name>.md
    skills/<name>/SKILL.md      ← thin Copilot wrapper → links to skills/<name>/SKILL.md

  agents/<name>.md              ← Claude Code adapter (also canonical for transition)
  plugin.json                   ← Claude Code metadata (skills, agents declarations)
```

**Dependency direction**: `adapters → shared core`. Never the reverse.

---

## What Lives Where

| Content type | Canonical location | Adapters may contain |
|---|---|---|
| Workflow logic (steps, guardrails, output format) | `skills/<name>/SKILL.md` | Runtime frontmatter + link to canonical |
| Agent persona and instructions | `agents/<name>.md` | Runtime metadata + mission summary + link |
| Engineering references | `references/**/*.md` | Mention by reference only — never copy prose |
| Shared workspace rules | `.github/copilot-instructions.md` | Referenced — not restated |
| Runtime metadata | `plugin.json` (Claude), `.github/` (Copilot) | Owned here; not in shared core |

---

## Adapter Contract

A valid adapter is intentionally thin. It contains only:

1. **Runtime frontmatter** — YAML metadata required by the runtime (name, description, tools, model)
2. **Short mission/when-to-use** — one paragraph max
3. **Links to canonical sources** — skill file, agent file, relevant references

Example — valid Copilot agent adapter (`.github/agents/reviewer.agent.md`):
```markdown
---
name: reviewer
description: Code review for production risk, architecture alignment, and correctness.
tools:
  - Read
  - Glob
  - Grep
---

Reviews pull requests and diffs for production risk, correctness, and architecture alignment.

**Canonical instructions**: [agents/reviewer.md](../../agents/reviewer.md)
**References used**: [practices/api-design.md](../../references/practices/api-design.md)
```

An adapter becomes invalid when it:
- Duplicates behavioral instructions from the canonical file
- Adds constraints or guardrails not present in the canonical
- Grows beyond 30–40 lines of non-frontmatter content

When a wrapper starts growing, move the shared content back to the canonical file.

---

## Authoring Rules

### Adding a new skill
1. Create `skills/<name>/SKILL.md` with the full workflow
2. Create `.github/skills/<name>/SKILL.md` as a thin wrapper with frontmatter + link
3. Declare the skill in `plugin.json` for Claude Code discovery

### Adding a new agent
1. Create `agents/<name>.md` with the full persona and instructions
2. Create `.github/agents/<name>.agent.md` as a thin wrapper with frontmatter + mission + link
3. Keep tool declarations in sync between both adapters

### Updating existing behavior
1. Edit the canonical file (`skills/` or `agents/`)
2. Verify adapter wrappers still link correctly — update link text if sections moved
3. Do NOT propagate behavioral changes to adapter files

### Adding a new reference
1. Create `references/<category>/<name>.md`
2. Reference it by path from the relevant skill or agent — never copy its content
3. Declare it in `.specify/stack.yml` under `references.active` if it should be globally available

---

## Frontmatter Compatibility

GitHub Copilot and Claude Code have different frontmatter schemas. The canonical files use no frontmatter — clean Markdown only. Frontmatter belongs exclusively in adapter files.

| Field | GitHub Copilot | Claude Code / plugin.json |
|---|---|---|
| `name` | Required in SKILL.md frontmatter | Declared in `plugin.json` |
| `description` | Keyword-rich (used for skill discovery) | Trigger sentence (used by model) |
| `tools` | Declared per agent in `.agent.md` | Declared in `plugin.json` or agent file |
| `model` | Not declared at skill level | Optional in agent file |

Keep `description` fields keyword-rich for Copilot (search-optimized) and trigger-sentence style for Claude (model-optimized). These are different and should not be copy-pasted between adapters.

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Behavioral instructions in adapter file | Second source of truth; will drift |
| References prose copied into adapter | Duplication; changes in one won't sync to other |
| Canonical file has runtime frontmatter | Couples shared core to a specific runtime |
| `plugin.json` and `.github/` declare different tools for same agent | Capabilities diverge per runtime |
| Adapter grown to 100+ lines | Adapter has become a canonical; refactor |
| New skill only in one adapter | Feature unavailable on the other runtime |

---

## Review Criteria

- [ ] Logic changes made only in canonical files (`skills/`, `agents/`, `references/`)
- [ ] Adapter files contain only: frontmatter, short mission, links
- [ ] No prose from canonical files duplicated in adapters
- [ ] New skills have a canonical file AND adapters for both runtimes
- [ ] New agents declared in both `plugin.json` and `.github/agents/`
- [ ] `description` fields optimized for their respective runtime discovery model
- [ ] Dependency direction flows adapters → shared core, never the reverse
