---
name: docs-sync
description: Detects the project stack, loads relevant knowledge and agents, then synchronizes ./specify/docs with current specs and codebase state. Triggered when the user wants to update documentation, sync docs after code changes, initialize docs for a new project, or audit doc accuracy.
argument-hint: "[scope: init, index, architecture, integrations, or all]"
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---

# Docs Sync

## Objective

Detect the project stack, activate the appropriate knowledge and agents for that stack, then update `./specify/docs` to accurately reflect the current state of specs and code. Documentation must be derived from what exists — not invented.

---

## Phase 0 — Stack Detection (always run first)

Before touching any documentation, detect the project stack by reading:

```
package.json          → Node.js, dependencies, devDependencies
go.mod                → Go modules
pom.xml               → Java/Maven
build.gradle          → Java/Gradle
requirements.txt      → Python
pyproject.toml        → Python (uv/poetry)
Cargo.toml            → Rust
```

For Node.js projects, inspect `dependencies` and `devDependencies` in `package.json`:

| Dependency | Stack Signal |
|---|---|
| `typescript` | TypeScript |
| `@nestjs/core` | NestJS |
| `react`, `react-dom` | React |
| `next` | Next.js |
| `@prisma/client` | Prisma |
| `axios` | Axios |
| `fastify` | Fastify |
| `express` | Express |
| `zod` | Zod validation |
| `@tanstack/react-query` | TanStack Query |

Persist the detected stack to `./specify/docs/stack.md` (create or overwrite):

```markdown
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---

# Detected Stack

## Runtime
- [e.g., Node.js 20 / Go 1.22]

## Languages
- [TypeScript / Go / Python / etc.]

## Frameworks
- [NestJS / React / Next.js / Gin / etc.]

## Libraries
- [Prisma / Axios / TanStack Query / etc.]

## Active Agents
- [List from agent mapping below]

## Active Knowledge
- [List from knowledge mapping below]
```

After writing `stack.md`, if `./specify/docs/index.md` exists, ensure it has a `[stack.md](stack.md)` entry in the `## Derived Documentation` section. Add it if missing — do not duplicate if already present.

---

## Phase 1 — Knowledge Activation

Based on detected stack, load the relevant knowledge files before proceeding:

| Stack Signal | Knowledge File |
|---|---|
| TypeScript | `knowledge/languages/typescript.md` |
| Go | `knowledge/languages/go.md` |
| NestJS | `knowledge/frameworks/nestjs.md` |
| React / Next.js | `knowledge/frameworks/react.md` |
| Prisma | `knowledge/libs/prisma.md` |
| Axios | `knowledge/libs/axios.md` |
| Any | `knowledge/practices/hexagonal-architecture.md` |
| Any | `knowledge/utilities/error-handling.md` |
| Any | `knowledge/practices/documentation-derivation.md` |

Read all matching knowledge files. These inform how you interpret the codebase and what you document about it.

---

## Phase 2 — Agent Mapping

Based on detected stack, record the recommended agents in `./specify/docs/stack.md`:

| Stack Signal | Agent |
|---|---|
| NestJS / Express / Fastify / Go backend | `specify-sde:backend-architect` |
| Any backend | `specify-sde:reviewer` |
| Any | `specify-sde:debugger` |
| Any | `specify-sde:task-planner` |
| Any with `./specify/docs` | `specify-sde:docs-maintainer` |

Also write the agent list to `./specify/docs/index.md` under a `## Engineering Agents` section (create if absent, update if present):

```markdown
## Engineering Agents

Agents configured for this project's stack:

| Agent | Role |
|---|---|
| `specify-sde:backend-architect` | Backend design — APIs, persistence, integrations |
| `specify-sde:reviewer` | Code review and production risk assessment |
| `specify-sde:debugger` | Root cause analysis and failure investigation |
| `specify-sde:task-planner` | Spec-to-task breakdown and delivery planning |
| `specify-sde:docs-maintainer` | Documentation accuracy and currency |
```

Include only agents relevant to the detected stack.

---

## Phase 3 — Docs Sync (scope-dependent)

### `init`
Create `./specify/docs/` if it doesn't exist. Copy and fill templates from `specify-sde/templates/`:
- `specify-docs-index-template.md` → `./specify/docs/index.md`
- `specify-architecture-template.md` → `./specify/docs/architecture.md`
- `specify-integrations-template.md` → `./specify/docs/integrations.md`
- `specify-operations-template.md` → `./specify/docs/operations.md`
- Create `./specify/docs/decisions/` directory

Fill templates with real data extracted from the codebase and `./specify/specs`. Do not leave template placeholders — remove sections that have no data yet.

### `index`
Update `./specify/docs/index.md`:
- Project summary (grep README, main entry point, or ask)
- Stack section (from detected stack)
- Integrations list (grep for external service clients)
- Main features (from `./specify/specs`)
- Engineering Agents section (from Phase 2)
- `## Derived Documentation` section: must contain one link per file present in `./specify/docs/` (e.g., `stack.md`, `architecture.md`, `integrations.md`, `operations.md`). Add missing links; do not remove existing ones unless the target file no longer exists.

### `architecture`
Update `./specify/docs/architecture.md`:
- Component map (from codebase structure)
- New ADR if a decision was made

### `integrations`
Update `./specify/docs/integrations.md`:
- Add/update entries for detected integration clients (axios instances, SDK clients)
- Remove decommissioned integrations

### `all`
Run all scopes in order: stack detection → knowledge activation → agent mapping → index → architecture → integrations.

---

## Sync Rules (all scopes)

- Read the target doc before editing — never overwrite without reading
- Verify every claim against `./specify/specs` or code before writing
- Preserve existing correct structure — do not reorganize
- Do not write speculative content — hypotheses go in `index.md` under "Hypotheses & Pending Items"
- Minimal change: update only what has changed or is missing

---

## Output Format

```
## Docs Sync Report

### Stack Detected
[List of detected technologies]

### Knowledge Activated
[List of knowledge files read]

### Agents Configured
[List of agents written to docs]

### Updated
- [file]: [what was changed and why]

### Verified Accurate
- [file]: [section confirmed correct]

### Skipped
- [file]: [reason]

### Gaps Found
- [what couldn't be filled and why]
```

---

## Quality Bar

A docs sync is complete when:
- `./specify/docs/stack.md` exists and reflects current dependencies
- `./specify/docs/index.md` has an accurate `## Engineering Agents` section
- `./specify/docs/index.md` `## Derived Documentation` section links to every file present in `./specify/docs/` (including `stack.md` and `architecture.md`)
- All updated content is traceable to specs or code
- No speculative content was added
- Gaps are reported, not silently skipped

---

## Guardrails

- Never write content not backed by evidence from specs or code
- Never reorganize docs structure without explicit instruction
- Never delete content without confirming it is no longer accurate
- Do not add "planned" features as if they are implemented
- Do not use the words "robust", "scalable", "enterprise-grade" — describe facts

---

## Example

User: "Sync docs for this project"

Actions:
1. Read `package.json` → detects TypeScript, NestJS, Prisma, Axios
2. Load `knowledge/languages/typescript.md`, `knowledge/frameworks/nestjs.md`, `knowledge/libs/prisma.md`, `knowledge/libs/axios.md`
3. Write `./specify/docs/stack.md` with detected stack + agent mapping
4. Update `./specify/docs/index.md` — stack section + `## Engineering Agents`
5. Update `./specify/docs/integrations.md` — add detected Axios instances
6. Report: stack detected, 4 knowledge files loaded, 4 agents configured, 2 docs updated
