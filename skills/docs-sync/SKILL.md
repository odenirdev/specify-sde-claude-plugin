---
name: docs-sync
description: Detects the project stack, loads relevant knowledge and agents, then synchronizes ./.specify/docs with current specs and codebase state. Triggered when the user wants to update documentation, sync docs after code changes, initialize docs for a new project, or audit doc accuracy.
argument-hint: "[scope: init, index, architecture, integrations, or all]"
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---

# Docs Sync

## Objective

Detect the project stack, activate the appropriate knowledge and agents for that stack, then update `./.specify/docs` to accurately reflect the current state of specs and code. Documentation must be derived from what exists — not invented.

---

## Phase 0 — Stack Detection (always run first)

Read `references/stack-signals.md` for the full list of files to inspect and dependency signals to match.

Persist the detected stack to `./.specify/docs/stack.md` (create or overwrite):

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

After writing `stack.md`, if `./.specify/docs/index.md` exists, ensure it has a `[stack.md](stack.md)` entry in the `## Derived Documentation` section. Add if missing — do not duplicate if already present.

---

## Monorepo Mode (detect after Phase 0)

**Detection**: monorepo mode is active if any of the following exist:
- `turbo.json` at the repository root
- `pnpm-workspace.yaml` at the repository root
- `package.json` root contains a `"workspaces"` field

When monorepo mode is active:

1. **List packages**: read `pnpm-workspace.yaml` or `package.json#workspaces` to enumerate all package paths.
2. **Apply `.specify` convention**:
   - Specs live **only** at `./.specify/specs/` (monorepo root) — never inside packages.
   - Root `./.specify/docs/` covers the monorepo as a whole (architecture, ADRs, cross-cutting concerns).
   - For each package: write or update `./<package>/.specify/docs/` with package-specific documentation (stack, architecture, public API). Never create `./<package>/.specify/specs/`.
3. **Report mode** in the output under `### Mode: Monorepo` with the list of detected packages.

**Directory layout enforced:**
```
<root>/
  .specify/
    specs/          ← only location for all specs
    docs/           ← monorepo-level docs
  <package>/
    .specify/
      docs/         ← package-specific docs only
      # NEVER specs/ here
```

---

## Phase 1 — Knowledge Activation

Read `references/knowledge-map.md` for the full stack-to-knowledge-file mapping.

Load all matching knowledge files. These inform how you interpret the codebase and what you document about it.

---

## Phase 2 — Agent Mapping

Read `references/agent-map.md` for the full stack-to-agent mapping.

Write the relevant **runtime-neutral** agent roles to `./.specify/docs/stack.md` under `## Active Agents`. Also write or update the `## Engineering Agents` section in `./.specify/docs/index.md`:

```markdown
## Engineering Agents

Primary agent roles configured for this project's stack:

| Agent | Role |
|---|---|
| `backend-architect` | Backend design — APIs, persistence, integrations |
| `reviewer` | Code review and production risk assessment |
| `debugger` | Root cause analysis and failure investigation |
| `task-planner` | Spec-to-task breakdown and delivery planning |
| `docs-maintainer` | Documentation accuracy and currency |
```

Use the generic role names in docs even if a particular runtime adapter prefixes them differently. Include only agents relevant to the detected stack.

---

## Phase 3 — Docs Sync (scope-dependent)

### `init`

Create `./.specify/docs/` if it doesn't exist.

Read `references/templates.md` for the full template-to-target mapping and per-template notes.

Fill each template with real data extracted from the codebase and `./.specify/specs`. Do not leave template placeholders — remove sections that have no data yet.

After filling templates, verify `./.specify/docs/index.md` has `[stack.md](./stack.md)` in the `## Derived Documentation` section — add it if missing.

### `index`

Update `./.specify/docs/index.md`:
- Project summary (grep README, main entry point, or ask)
- Stack section (from detected stack)
- Integrations list (grep for external service clients)
- Main features (from `./.specify/specs`)
- Engineering Agents section (from Phase 2)
- `## Derived Documentation` section: must contain one link per file present in `./.specify/docs/` (e.g., `stack.md`, `architecture.md`, `integrations.md`, `operations.md`). Add missing links; do not remove existing ones unless the target file no longer exists.

### `architecture`

Update `./.specify/docs/architecture.md`:
- Component map (from codebase structure)
- New ADR if a decision was made

### `integrations`

Update `./.specify/docs/integrations.md`:
- Add/update entries for detected integration clients (axios instances, SDK clients)
- Remove decommissioned integrations

### `all`

Run all scopes in order: stack detection → knowledge activation → agent mapping → index → architecture → integrations.

---

## Sync Rules (all scopes)

- Read the target doc before editing — never overwrite without reading
- Verify every claim against `./.specify/specs` or code before writing
- Preserve existing correct structure — do not reorganize
- Do not write speculative content — hypotheses go in `index.md` under "Hypotheses & Pending Items"
- Minimal change: update only what has changed or is missing

---

<output_format>
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
</output_format>

---

## Quality Bar

A docs sync is complete when:
- `./.specify/docs/stack.md` exists and reflects current dependencies
- `./.specify/docs/index.md` has an accurate `## Engineering Agents` section
- `./.specify/docs/index.md` `## Derived Documentation` section links to every file present in `./.specify/docs/` (including `stack.md` and `architecture.md`)
- All updated content is traceable to specs or code
- No speculative content was added
- Gaps are reported, not silently skipped

---

## Knowledge to Consult

- `knowledge/practices/documentation-derivation.md` — writing accurate, evidence-based documentation

## Guardrails

- Never write content not backed by evidence from specs or code
- Never reorganize docs structure without explicit instruction
- Never delete content without confirming it is no longer accurate
- Do not add "planned" features as if they are implemented
- Do not use the words "robust", "scalable", "enterprise-grade" — describe facts

---

<examples>
<example>
<user.prompt>Sync docs for this project</user.prompt>
<context>TypeScript/NestJS project with Prisma and Axios. No ./.specify/docs yet.</context>
<actions>
1. Read `package.json` → detects TypeScript, NestJS, Prisma, Axios
2. Read `references/stack-signals.md` → confirm dependency signals
3. Write `./.specify/docs/stack.md` with detected stack
4. Read `references/knowledge-map.md` → load typescript.md, nestjs.md, prisma.md, axios.md
5. Read `references/agent-map.md` → map backend-architect, reviewer, debugger, task-planner, docs-maintainer
6. Update `./.specify/docs/stack.md` with Active Agents list
7. Update `./.specify/docs/index.md` — stack section + `## Engineering Agents`
8. Update `./.specify/docs/integrations.md` — add detected Axios instances
9. Report: stack detected, 4 knowledge files loaded, 5 agents configured, 2 docs updated
</actions>
</example>
<example>
<user.prompt>Init docs for a new Go project</user.prompt>
<context>Go backend with go.mod. No ./.specify/docs directory.</context>
<actions>
1. Read `go.mod` → detects Go modules and module path
2. Read `references/stack-signals.md` → confirm Go stack
3. Write `./.specify/docs/stack.md` with Go runtime
4. Read `references/knowledge-map.md` → load go.md, hexagonal-architecture.md, error-handling.md, documentation-derivation.md
5. Read `references/agent-map.md` → map backend-architect, reviewer, debugger, task-planner, docs-maintainer
6. Read `references/templates.md` → get template-to-target mapping
7. Read each template, fill with real data, write to ./.specify/docs/
8. Create ./.specify/docs/decisions/ directory
9. Report: Go stack detected, 4 knowledge files loaded, 5 agents configured, 4 docs created
</actions>
</example>
<example>
<user.prompt>Sync docs for this monorepo</user.prompt>
<context>TypeScript monorepo with Turborepo and pnpm workspaces. Packages: apps/web (Next.js), apps/api (NestJS), packages/ui.</context>
<actions>
1. Read `turbo.json` and `pnpm-workspace.yaml` → monorepo mode active; packages: apps/web, apps/api, packages/ui
2. Read `package.json` at root → detects TypeScript, turbo
3. Read `references/stack-signals.md` → confirm Turborepo + monorepo + TypeScript signals
4. Write `./.specify/docs/stack.md` with detected stack + "### Mode: Monorepo" section listing all packages
5. Read `references/knowledge-map.md` → load turborepo.md, monorepo.md, typescript.md, hexagonal-architecture.md
6. Read `references/agent-map.md` → map all relevant agents
7. Update root `./.specify/docs/index.md` and `./.specify/docs/architecture.md` with monorepo-level data
8. For apps/web: detect Next.js, write `apps/web/.specify/docs/stack.md` and `apps/web/.specify/docs/index.md` — no specs/ created
9. For apps/api: detect NestJS + Prisma, write `apps/api/.specify/docs/stack.md` and `apps/api/.specify/docs/index.md` — no specs/ created
10. For packages/ui: detect React, write `packages/ui/.specify/docs/stack.md` — no specs/ created
11. Verify `.specify/specs/` exists only at root — warn if found inside any package
12. Report: Monorepo mode, 3 packages documented, root + 3 package docs updated, 0 specs/ created in packages
</actions>
</example>
</examples>
