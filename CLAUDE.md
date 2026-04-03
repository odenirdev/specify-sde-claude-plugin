# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repository Is

`specify-sde` is a Claude Code plugin — a collection of skills, agents, knowledge references, and templates that extend Claude Code's engineering capabilities. There is no runtime code, build system, or test suite. The entire content is Markdown.

---

## Plugin Structure

```
plugin.json                  — Plugin manifest (name, version, description)
skills/<name>/SKILL.md       — Reusable engineering capabilities (loaded by Claude on trigger)
agents/<name>.md             — Specialized roles that compose skills
knowledge/<category>/<name>.md  — Stack-aware engineering references
templates/<name>.md          — Starting structures for ./specify/docs and project conventions
```

---

## Authoring Rules

### Skills (`skills/*/SKILL.md`)
Each skill requires YAML frontmatter:
```yaml
---
name: skill-name
description: [Third-person. Specific trigger phrases. What it does and when.]
argument-hint: "[what the user passes]"
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---
```

Skill body structure (in order):
1. **Objective** — what it produces
2. **When to Use** — concrete scenarios
3. **Inputs** — what it expects
4. **Responsibilities** — numbered phases, each with explicit sub-steps
5. **Output Location** — where to write (`./specify/specs/<slug>/<type>.md`) with `updated_at` frontmatter
6. **Output Format** — exact markdown structure the skill must produce
7. **Quality Bar** — checklist for "done"
8. **Knowledge to Consult** — paths to `knowledge/` files
9. **Repository Areas to Inspect** — where to read in the consuming project
10. **Guardrails** — explicit prohibitions
11. **Example** — concrete user prompt + step-by-step actions

Skills that write spec artifacts (`definition.md`, `architecture.md`, `tasks.md`, `tradeoff.md`) must include this frontmatter in their output files:
```
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---
```
If the file exists, overwrite and refresh `updated_at`. Do not append.

### Agents (`agents/*.md`)
Each agent requires YAML frontmatter:
```yaml
---
name: specify-sde:<agent-name>
description: [Third-person. Role + when to trigger.]
tools: Read, Glob, Grep, Bash, Write, Edit
model: claude-opus-4-6 | claude-sonnet-4-6
color: red | blue | yellow | green | purple
---
```

Agent body structure:
- **Role** — one-line job title
- **Mission** — what it optimizes for
- **When to Use** — scenarios
- `<examples>` block with 2–3 `<example>` entries (user prompt + context)
- **Skills Used** — which skills it invokes
- **Knowledge to Prioritize** — relevant `knowledge/` paths
- **Constraints** — explicit behavioural limits
- **Output Style** — format and structure of its response

### Knowledge (`knowledge/**/*.md`)
Each file covers one technology or practice. Required sections:
- **Best Practices** — concrete, specific guidance
- **Anti-Patterns** — table of pattern → problem → fix
- **Review Criteria** — checklist for code review
- **Trade-offs** — decision points with explicit pros/cons
- **Implementation Notes** — environment setup, tooling, gotchas

No generic filler. Every line must be actionable.

---

## Spec and Docs Integration

Skills operate on two directories in the **consuming project** (not this repo):

- `./specify/specs/` — source of truth; skills read from and write to `<slug>/<type>.md`
- `./specify/docs/` — derived documentation; `docs-sync` maintains it

The `docs-sync` skill has a special responsibility: detect the consuming project's stack (via `package.json`, `go.mod`, etc.), load matching knowledge files, and write `./specify/docs/stack.md` with the active agents and knowledge.

---

## Adding Content

**New knowledge file**: add to the matching `knowledge/` subdirectory. Follow the five-section structure above. Reference it in any skill or agent where it is relevant under "Knowledge to Consult" / "Knowledge to Prioritize".

**New skill**: create `skills/<name>/SKILL.md`. Add it to the relevant agent's "Skills Used" section if applicable.

**New agent**: create `agents/<name>.md`. Ensure the `<examples>` block has at least 2 concrete entries — vague examples produce poor triggering.

**New template**: add to `templates/`. Reference it in `docs-sync` skill under the `init` scope.
