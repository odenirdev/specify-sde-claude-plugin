---
name: knowledge-update
description: Guides the user to create or update a knowledge file in the plugin following the standard template pattern. Triggered when the user wants to add a new framework, library, language, utility, or practice to the plugin's knowledge base.
argument-hint: "[category: frameworks|languages|libs|utilities|practices] [technology name]"
allowed-tools: Read, Glob, Grep, Write, Edit
---

# Knowledge Update

## Objective

Guide the creation or update of a `knowledge/<category>/<name>.md` file following the plugin's standard five-section structure. The output is an actionable, opinionated engineering reference — no generic content.

---

## When to Use

Use this skill when the user wants to:
- Add a new framework, library, language, utility, or engineering practice to the knowledge base
- Update an existing knowledge file that is outdated or incomplete
- Audit whether an existing knowledge file follows the standard structure

---

## Inputs

- **Category**: one of `frameworks`, `languages`, `libs`, `utilities`, `practices`
- **Name**: the technology or practice (e.g., `nestjs`, `prisma`, `go`, `testing`)
- (Optional) Existing knowledge from the current conversation or user-provided content

---

## Responsibilities

### Phase 1 — Identify Category and Load Template

1. Determine the correct category from the argument or by asking the user:
   - `frameworks` — web/backend frameworks (NestJS, React, Next.js, Gin, FastAPI)
   - `languages` — programming languages (TypeScript, Go, Python, Java)
   - `libs` — specific libraries and ORMs (Prisma, Axios, TanStack Query)
   - `utilities` — cross-cutting concerns (error-handling, logging, testing)
   - `practices` — engineering patterns (api-design, hexagonal-architecture, task-breakdown)

2. Load the matching template from `references/`:
   - `frameworks` → `references/framework-template.md`
   - `languages` → `references/language-template.md`
   - `libs` → `references/lib-template.md`
   - `utilities` → `references/utility-template.md`
   - `practices` → `references/practice-template.md`

3. Check if a file already exists at `knowledge/<category>/<name>.md`. If it exists, read it and plan to update rather than overwrite.

### Phase 2 — Populate the Knowledge File

Fill every section of the template with content that is:
- **Specific and actionable** — no generic platitudes
- **Opinionated** — state what the correct approach is, not "it depends"
- **Based on real patterns** — reference actual APIs, decorators, flags, or tools

Follow the five mandatory sections from the template. Do not skip any section. Do not add sections not in the template.

### Phase 3 — Write the File

Write to `knowledge/<category>/<name>.md`.

Use this header format:
```
# [TechnologyName] — Engineering Reference
```

If the file already existed, overwrite it. Never append to an existing file.

### Phase 4 — Update the Knowledge Map

After writing, check `skills/docs-sync/references/knowledge-map.md`.

If the new file is not already listed, add a row:

```
| [Stack Signal] | `knowledge/<category>/<name>.md` |
```

Use the technology name as the stack signal (e.g., `Prisma`, `React`, `Go`).

---

## Output Location

```
knowledge/<category>/<name>.md
```

---

## Output Format

Every knowledge file must follow this exact structure (load category-specific template for details):

```markdown
# [Name] — Engineering Reference

## Best Practices

[Subsections with concrete, opinionated guidance]

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| ... | ... | ... |

---

## Review Criteria

- [ ] ...

---

## Trade-offs

**[Decision A vs B]**: [Recommendation with rationale]

---

## Implementation Notes

- ...
```

---

## Quality Bar

- [ ] All five sections present and populated
- [ ] No placeholder text (e.g., "[add content here]")
- [ ] Anti-patterns table has at least 3 rows
- [ ] Review criteria checklist has at least 5 items
- [ ] Trade-offs names a specific decision, not a generic topic
- [ ] Implementation notes reference real tools, commands, or flags
- [ ] knowledge-map.md updated if file is new

---

## Knowledge to Consult

- `references/framework-template.md` — for framework files
- `references/language-template.md` — for language files
- `references/lib-template.md` — for library files
- `references/utility-template.md` — for utility files
- `references/practice-template.md` — for practice files
- `skills/docs-sync/references/knowledge-map.md` — map to update

---

## Guardrails

- Do NOT create a new category outside of `frameworks`, `languages`, `libs`, `utilities`, `practices`
- Do NOT write generic content — every line must be actionable
- Do NOT add sections beyond the five defined in the template
- Do NOT leave placeholder text in the output file
- Do NOT modify knowledge files outside the target category/name
