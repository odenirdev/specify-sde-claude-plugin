---
name: knowledge-link
description: Audits and updates the "Knowledge to Consult" section in one or all skill SKILL.md files. Triggered when the user wants to sync knowledge references after adding new knowledge files, review which knowledge a skill uses, or update a specific skill's knowledge pointers.
argument-hint: "[skill-name | all]"
allowed-tools: Read, Glob, Grep, Write, Edit
---

# Knowledge Link

## Objective

Keep the "Knowledge to Consult" section of each `skills/*/SKILL.md` accurate and complete. Read the skill's purpose, scan existing knowledge files, determine relevance, and update the section in-place.

---

## When to Use

Use this skill when the user wants to:
- Sync knowledge references after adding new files via `knowledge-update`
- Update a specific skill's "Knowledge to Consult" section
- Audit all skills for missing or stale knowledge links

---

## Inputs

- **Skill name** (e.g., `engineer-review`) or `all` to process every skill
- Existing `skills/*/SKILL.md` files
- All files under `knowledge/`

---

## Responsibilities

### Phase 0 — Mode Detection

Check if `plugin.json` exists in the current working directory.

- **Plugin mode**: `plugin.json` found → proceed to Phase 1 (skill file audit).
- **Project mode**: `plugin.json` not found → skip to Phase P1 (project Active Knowledge update).

---

### PLUGIN MODE

### Phase 1 — Collect Available Knowledge

1. Glob all files matching `knowledge/**/*.md`.
2. For each file, read its first heading (`# [Name] — Engineering Reference`) and the first line of each section to extract what it covers.
3. Build an index:

```
knowledge/frameworks/nestjs.md      → NestJS, modules, DI, controllers, DTOs
knowledge/frameworks/react.md       → React, components, hooks, state
knowledge/languages/typescript.md   → TypeScript, strict mode, types
knowledge/languages/go.md           → Go, errors, goroutines
knowledge/libs/prisma.md            → Prisma, ORM, migrations, queries
knowledge/libs/axios.md             → Axios, HTTP client, interceptors
knowledge/utilities/error-handling.md → errors, exceptions, domain errors
knowledge/utilities/logging.md      → logging, structured logs, tracing
knowledge/utilities/testing.md      → tests, mocks, coverage
knowledge/practices/hexagonal-architecture.md → ports, adapters, domain, layers
knowledge/practices/api-design.md   → REST, API contracts, versioning
knowledge/practices/task-breakdown.md → tasks, phases, acceptance criteria
knowledge/practices/documentation-derivation.md → docs, derivation, accuracy
```

### Phase 2 — Resolve Target Skills

- If argument is a specific skill name (e.g., `engineer-review`): process only `skills/engineer-review/SKILL.md`.
- If argument is `all` or omitted: glob `skills/*/SKILL.md` and process each one.

### Phase 3 — Analyze Each Skill

For each target skill:

1. Read the full `SKILL.md`.
2. Extract the skill's **Objective**, **When to Use**, and **Responsibilities** sections to understand what it does.
3. For each knowledge file in the index, assess relevance:
   - **High relevance**: the skill explicitly works with the domain this knowledge covers (e.g., a review skill is always relevant to error-handling, testing, and architecture)
   - **Conditional relevance**: the knowledge applies only when the codebase uses a specific technology (e.g., `prisma.md` is relevant to review only if the project uses Prisma)
   - **Not relevant**: no overlap between skill purpose and knowledge domain

4. Build the updated "Knowledge to Consult" list:
   - Always include: knowledge files that apply to all projects (practices, utilities)
   - Mark as conditional: framework/language/lib knowledge — add a note `(load if detected in stack)`
   - Exclude: knowledge files with no overlap

### Phase 4 — Update the Section

1. Locate the existing `## Knowledge to Consult` section in the skill file.
2. Replace its content with the updated list. Format:

```markdown
## Knowledge to Consult

- `knowledge/practices/hexagonal-architecture.md` — architecture rules
- `knowledge/utilities/error-handling.md` — error handling patterns
- `knowledge/utilities/testing.md` — test coverage expectations
- `knowledge/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `knowledge/languages/typescript.md` — TypeScript patterns (load if detected in stack)
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

### Phase P2 — Load Knowledge Map

Read `skills/docs-sync/references/knowledge-map.md` from the plugin directory (the directory where this skill's `SKILL.md` lives, two levels up from `skills/knowledge-link/`).

Map each detected technology to its knowledge file using the table in `knowledge-map.md`. Always include files marked `Any`.

### Phase P3 — Update Active Knowledge in stack.md

1. Read `./.specify/docs/stack.md`. If the file does not exist, stop and report: "`./.specify/docs/stack.md` not found — run `docs-sync init` first."
2. Locate the `## Active Knowledge` section.
3. Replace its content with the matched knowledge file paths. Format:

```markdown
## Active Knowledge

- knowledge/languages/typescript.md
- knowledge/frameworks/react.md
- knowledge/practices/hexagonal-architecture.md
- knowledge/utilities/error-handling.md
- knowledge/practices/documentation-derivation.md
```

4. Do NOT modify any other section of `stack.md`.
5. Do NOT add paths for knowledge files not present in the plugin's `knowledge/` directory — verify each path exists before writing.

### Phase P4 — Report

Output a summary:

```
## Active Knowledge Updated

Stack detected: TypeScript, React
Knowledge files linked: 5
File updated: ./.specify/docs/stack.md
```

---

## Output Location

- **Plugin mode**: in-place edit of `skills/<name>/SKILL.md` — only the `## Knowledge to Consult` section.
- **Project mode**: in-place edit of `./.specify/docs/stack.md` — only the `## Active Knowledge` section.

---

## Quality Bar

**Plugin mode:**
- [ ] No knowledge file listed that does not exist at the path shown
- [ ] Conditional references are marked `(load if detected in stack)`
- [ ] Universal knowledge (practices, utilities) is always included for applicable skills
- [ ] No other section of the skill file was modified
- [ ] Paths use the form `knowledge/<category>/<name>.md` — no absolute paths

**Project mode:**
- [ ] Mode was correctly detected (no `plugin.json` in project root)
- [ ] Stack was detected from actual project files — not assumed
- [ ] Every listed knowledge path was verified to exist in the plugin before writing
- [ ] Only `## Active Knowledge` was modified in `stack.md`
- [ ] Paths use the form `knowledge/<category>/<name>.md` — no absolute paths

---

## Guardrails

- Do NOT rewrite or refactor any other section of the skill file or `stack.md`
- Do NOT add knowledge files that do not exist on disk
- Do NOT remove a knowledge reference without a clear reason (e.g., file deleted or zero overlap)
- Do NOT create new knowledge files — use the `knowledge-update` skill for that
- In project mode, do NOT run the plugin-mode phases — they are mutually exclusive
- In project mode, if `./.specify/docs/stack.md` is missing, stop and instruct the user to run `docs-sync init` first
