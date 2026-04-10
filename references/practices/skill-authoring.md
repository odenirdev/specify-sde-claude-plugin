# Skill Authoring — Engineering Reference

## Core Principle

A skill is a reusable, self-contained workflow exposed to an AI agent. The agent reads SKILL.md only when the skill becomes relevant — every token in that file competes with conversation history and tool results. Write for precision and context efficiency, not exhaustiveness.

## SKILL.md Structure

Every skill requires a SKILL.md with two parts:

```markdown
---
name: skill-name
description: One-line trigger description — what this skill does and when to use it.
---

# Skill Name

## Objective
What the skill produces or decides.

## When to Use
Exact trigger conditions — explicit or pattern-based.

## Inputs
Optional flags, arguments, or context the skill consumes.

## Responsibilities
Step-by-step instructions. Use numbered steps for sequential flows.

## Output Format
The exact shape of what the skill produces (markdown, YAML, etc.).

## Guardrails
What the skill must refuse, escalate, or warn about.
```

### Frontmatter Rules
- `name`: kebab-case, matches the directory name
- `description`: written as a trigger sentence — Claude uses this to decide when to load the skill; make it unambiguous and specific

---

## Progressive Disclosure

Load only what is needed for the current step. Structure your skill around this principle:

1. **Frontmatter** — minimal: name + description. Claude reads this to decide relevance.
2. **SKILL.md** — comprehensive but focused: loaded when skill is chosen. Every section must earn its place.
3. **Helper files** — loaded on demand: reference docs, templates, schemas. Link them from SKILL.md; do not inline.

Example: a form-filling skill references `forms.md` only when the step that fills forms is active — not upfront.

---

## Context Window Efficiency

- Keep SKILL.md under ~400 lines; split into helper files if larger
- Use tables and bullet lists — structure is parsed faster than prose
- Move examples and schemas to companion files (`examples/`, `templates/`)
- Avoid repeating instructions already in the system prompt or CLAUDE.md
- Do not include content that would never apply to the current runtime

---

## On-Demand File Loading

A skill can reference dozens of helper files. Claude loads only the files needed per task step. Design for this:

```
skills/my-skill/
  SKILL.md          ← always loaded when skill activates
  templates/
    output.md       ← loaded only when producing output
  references/
    schema.md       ← loaded only when validating schema
```

Reference from SKILL.md:
```markdown
See [output template](templates/output.md) when producing the final report.
```

---

## Iterative Improvement Workflow

1. **Write** — author the minimal SKILL.md with objective, steps, output format
2. **Test with Claude B** — run the skill on representative inputs as if Claude has no prior context
3. **Identify failures** — wrong output shape, wrong scope, missed guardrail
4. **Fix one thing at a time** — add the smallest rule that prevents the failure
5. **Re-test** — confirm the fix does not break prior outputs
6. **Promote** — when the skill handles all representative cases, mark it stable

Do not rewrite the entire SKILL.md when one step fails. Diagnose first.

---

## Adapter Wrappers

Skills live in the shared core (`skills/<name>/SKILL.md`). Runtime adapters are thin wrappers that reference the shared skill:

- **GitHub Copilot**: `.github/skills/<name>/SKILL.md` — copy or symlink, or reference the canonical file
- **Claude Code**: loaded via `plugin.json` skills declaration or direct path reference

Never duplicate skill logic in adapters. Adapters add only runtime-specific wiring (path references, tool declarations).

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Skill description is vague ("does things") | Claude loads it for unrelated tasks or never |
| All logic inlined in SKILL.md | Context bloat; harder to maintain |
| Guardrails missing | Agent takes risky actions without warning |
| Output format undefined | Shape varies per run; downstream tools break |
| Skill covers two unrelated workflows | Violates single responsibility; split it |
| No trigger conditions | Claude can't decide when to activate the skill |

---

## Review Criteria

- [ ] `name` matches the directory name exactly
- [ ] `description` reads as an unambiguous trigger sentence
- [ ] Objective is stated in one sentence
- [ ] Steps are sequential and completable in order
- [ ] Output format is explicit and verifiable
- [ ] Guardrails cover the highest-risk actions
- [ ] Helper files are referenced, not inlined
- [ ] SKILL.md stays under ~400 lines
- [ ] Adapter wrappers reference the shared core — no duplicated logic
