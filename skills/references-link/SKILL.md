---
name: references-link
description: Audits and updates the "References to Consult" section in one or all skill SKILL.md files. Triggered when the user wants to sync references after adding new reference files, review which references a skill uses, or update a specific skill's reference pointers.
argument-hint: "[skill-name | all]"
allowed-tools: Read, Glob, Grep, Write, Edit
---

# References Link

## Objective

Keep the "References to Consult" section of each `skills/*/SKILL.md` accurate and complete. Read the skill's purpose, scan existing reference files, determine relevance, and update the section in-place.

---

## When to Use

Use this skill when the user wants to:
- Sync references after adding new files via `references-update`
- Update a specific skill's "References to Consult" section
- Audit all skills for missing or stale reference links

---

## Inputs

- **Skill name** (e.g., `engineer-review`) or `all` to process every skill
- Existing `skills/*/SKILL.md` files
- All files under `references/`

---

## Responsibilities

### Phase 0 — Mode Detection

Check if `plugin.json` exists in the current working directory.

- **Plugin mode**: `plugin.json` found → proceed to Phase 1 (skill file audit).
- **Project mode**: `plugin.json` not found → skip to Phase P1 (project Active References update).

---

### PLUGIN MODE

### Phase 1 — Collect Available References

1. Glob all files matching `references/**/*.md`.
2. For each file, read its first heading (`# [Name] — Engineering Reference`) and the first line of each section to extract what it covers.
3. Build an index:

```
references/frameworks/nestjs.md      → NestJS, modules, DI, controllers, DTOs
references/frameworks/react.md       → React, components, hooks, state
references/languages/typescript.md   → TypeScript, strict mode, types
references/languages/go.md           → Go, errors, goroutines
references/libs/prisma.md            → Prisma, ORM, migrations, queries
references/libs/axios.md             → Axios, HTTP client, interceptors
references/utilities/error-handling.md → errors, exceptions, domain errors
references/utilities/logging.md      → logging, structured logs, tracing
references/utilities/testing.md      → tests, mocks, coverage
references/practices/hexagonal-architecture.md → ports, adapters, domain, layers
references/practices/api-design.md   → REST, API contracts, versioning
references/practices/task-breakdown.md → tasks, phases, acceptance criteria
references/practices/documentation-derivation.md → docs, derivation, accuracy
```

### Phase 2 — Resolve Target Skills

- If argument is a specific skill name (e.g., `engineer-review`): process only `skills/engineer-review/SKILL.md`.
- If argument is `all` or omitted: glob `skills/*/SKILL.md` and process each one.

### Phase 3 — Analyze Each Skill

For each target skill:

1. Read the full `SKILL.md`.
2. Extract the skill's **Objective**, **When to Use**, and **Responsibilities** sections to understand what it does.
3. For each reference file in the index, assess relevance:
   - **High relevance**: the skill explicitly works with the domain this reference covers (e.g., a review skill is always relevant to error-handling, testing, and architecture)
   - **Conditional relevance**: the reference applies only when the codebase uses a specific technology (e.g., `prisma.md` is relevant to review only if the project uses Prisma)
   - **Not relevant**: no overlap between skill purpose and reference domain

4. Build the updated "References to Consult" list:
   - Always include: reference files that apply to all projects (practices, utilities)
   - Mark as conditional: framework/language/lib references — add a note `(load if detected in stack)`
   - Exclude: reference files with no overlap

### Phase 4 — Update the Section

1. Locate the existing `## References to Consult` section in the skill file.
2. Replace its content with the updated list. Format:

```markdown
## References to Consult

- `references/practices/hexagonal-architecture.md` — architecture rules
- `references/utilities/error-handling.md` — error handling patterns
- `references/utilities/testing.md` — test coverage expectations
- `references/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `references/languages/typescript.md` — TypeScript patterns (load if detected in stack)
```

3. If the section does not exist in the file, add it before `## Guardrails` or at the end of the file.
4. Do NOT modify any other section of the skill file.

### Phase 5 — Report

After processing all target skills, output a summary table:

```
| Skill | Added | Removed | Unchanged |
|---|---|---|---|
| engineer-review | 2 | 0 | 3 |
| engineer-debug  | 1 | 1 | 2 |
```

---

### PROJECT MODE

### Phase P1 — Stack Detection

Inspect the project root for stack signals:

| File | Signal |
|---|---|
| `package.json` | Read `dependencies` and `devDependencies` |
| `go.mod` | Go project |
| `requirements.txt` / `pyproject.toml` | Python project |
| `pom.xml` / `build.gradle` | Java project |

Extract detected technologies (languages, frameworks, libs).

### Phase P2 — Load References Map

Read `skills/docs-sync/references/references-map.md` from the plugin directory (the directory where this skill's `SKILL.md` lives, two levels up from `skills/references-link/`).

Map each detected technology to its reference file using the table in `references-map.md`. Always include files marked `Any`.

### Phase P3 — Update Active References in stack.md

1. Read `./.specify/docs/stack.md`. If the file does not exist, stop and report: "`./.specify/docs/stack.md` not found — run `docs-sync init` first."
2. Locate the `## Active References` section.
3. Replace its content with the matched reference file paths. Format:

```markdown
## Active References

- references/languages/typescript.md
- references/frameworks/react.md
- references/practices/hexagonal-architecture.md
- references/utilities/error-handling.md
- references/practices/documentation-derivation.md
```

4. Do NOT modify any other section of `stack.md`.
5. Do NOT add paths for reference files not present in the plugin's `references/` directory — verify each path exists before writing.

### Phase P4 — Report

Output a summary:

```
## Active References Updated

Stack detected: TypeScript, React
Reference files linked: 5
File updated: ./.specify/docs/stack.md
```

---

## Output Location

- **Plugin mode**: in-place edit of `skills/<name>/SKILL.md` — only the `## References to Consult` section.
- **Project mode**: in-place edit of `./.specify/docs/stack.md` — only the `## Active References` section.

---

## Quality Bar

**Plugin mode:**
- [ ] No reference file listed that does not exist at the path shown
- [ ] Conditional references are marked `(load if detected in stack)`
- [ ] Universal references (practices, utilities) is always included for applicable skills
- [ ] No other section of the skill file was modified
- [ ] Paths use the form `references/<category>/<name>.md` — no absolute paths

**Project mode:**
- [ ] Mode was correctly detected (no `plugin.json` in project root)
- [ ] Stack was detected from actual project files — not assumed
- [ ] Every listed reference path was verified to exist in the plugin before writing
- [ ] Only `## Active References` was modified in `stack.md`
- [ ] Paths use the form `references/<category>/<name>.md` — no absolute paths

---

## Guardrails

- Do NOT rewrite or refactor any other section of the skill file or `stack.md`
- Do NOT add reference files that do not exist on disk
- Do NOT remove a reference entry without a clear reason (e.g., file deleted or zero overlap)
- Do NOT create new reference files — use the `references-update` skill for that
- In project mode, do NOT run the plugin-mode phases — they are mutually exclusive
- In project mode, if `./.specify/docs/stack.md` is missing, stop and instruct the user to run `docs-sync init` first
