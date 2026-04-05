---
name: docs-sync
description: Detects the project stack, loads relevant references and agents, then synchronizes `./.specify/docs`, `CLAUDE.md`, and managed `README.md` content with the current specs and codebase state. Triggered when the user wants to update documentation, sync docs after code changes, initialize docs for a new project, or audit doc accuracy.
argument-hint: "[scope: init, index, architecture, integrations, claude, readme, or all]"
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---

# Docs Sync

## Objective

Detect the project stack, activate the appropriate references and agents for that stack, then update `./.specify/docs` and the project's derived entrypoints to accurately reflect the current state of specs and code. Documentation must be derived from what exists — not invented.

## Entry Point Convention

`./.specify/docs` is the source of truth for engineering context. The other entrypoint files are derived from it and must stay intentionally thin:

- `./.specify/docs/` — detailed engineering context, architecture, operations, and ADRs
- `CLAUDE.md` — minimal bridge for agent runtimes and tools
- `README.md` — short human overview, getting started, and links

If `CLAUDE.md` or `README.md` disagree with `./.specify/docs`, `./.specify/docs` wins.

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

## Active References
- [List from references mapping below]
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
   - Create or validate `./<package>/CLAUDE.md` **only** when `./<package>/.specify/docs/` already exists or is created during this sync. Do not create package `README.md` files unless the user explicitly asks for them.
3. **Report mode** in the output under `### Mode: Monorepo` with the list of detected packages and which packages were eligible for package-level `CLAUDE.md` sync.

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

## Phase 1 — References Activation

Read `references/references-map.md` for the full stack-to-references mapping.

Load all matching reference files. These inform how you interpret the codebase and what you document about it.

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

During `init`, also:
- create or normalize the root `CLAUDE.md` as a minimal bridge to the derived docs;
- create `README.md` if it does not exist, or insert one managed block if it already exists without markers;
- in monorepo mode, create package-level `CLAUDE.md` files only for packages with local `./.specify/docs/`.

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

### `claude`

Update or create the root `CLAUDE.md`:
- Keep it minimal and link-oriented
- Point first to `./.specify/docs/index.md`
- Link to `.github/copilot-instructions.md` if present
- Link to `README.md` for quick orientation
- Never duplicate detailed rules, stack tables, or architecture prose

In monorepo mode:
- root `CLAUDE.md` points to the root docs;
- each eligible package `CLAUDE.md` points first to `./.specify/docs/index.md` inside that package, then to the root docs if needed;
- packages without local docs are skipped and reported.

### `readme`

Update or create the root `README.md`:
- Only manage the content between `<!-- docs-sync:start -->` and `<!-- docs-sync:end -->`
- If the markers do not exist, insert a single managed block and preserve all other content
- If the file does not exist, create a concise README with a short intro and one managed block
- Keep the managed block short and derived; it may include only:
  - project overview,
  - getting started,
  - main commands or workflows,
  - basic structure,
  - links to `./.specify/docs/*`
- Do not turn the README into full architecture docs; link outward instead.

### `all`

Run all scopes in order: stack detection → references activation → agent mapping → index → architecture → integrations → claude → readme.

---

## Sync Rules (all scopes)

- Read the target doc before editing — never overwrite without reading
- Verify every claim against `./.specify/specs` or code before writing
- Preserve existing correct structure — do not reorganize
- Do not write speculative content — hypotheses go in `index.md` under "Hypotheses & Pending Items"
- Treat `./.specify/docs` as the source of truth; `CLAUDE.md` and `README.md` are derived entrypoints only
- In `README.md`, edit only the content inside `<!-- docs-sync:start -->` and `<!-- docs-sync:end -->`; preserve all manual content outside the block
- Keep `CLAUDE.md` minimal and link-oriented — no duplicated rules, architecture walkthroughs, or stack tables
- Minimal change: update only what has changed or is missing

---

<output_format>
## Docs Sync Report

### Stack Detected
[List of detected technologies]

### References Activated
[List of reference files read]

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
- Root `CLAUDE.md` exists, stays minimal, and points to the derived docs
- Root `README.md` contains a short managed block that points readers to `./.specify/docs`
- In monorepo mode, only packages with local `./.specify/docs/` receive package-level `CLAUDE.md`
- Manual README content outside the managed block is preserved
- All updated content is traceable to specs or code
- No speculative content was added
- Gaps are reported, not silently skipped

---

## References to Consult

- `references/practices/documentation-derivation.md` — writing accurate, evidence-based documentation

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
4. Read `references/references-map.md` → load typescript.md, nestjs.md, prisma.md, axios.md
5. Read `references/agent-map.md` → map backend-architect, reviewer, debugger, task-planner, docs-maintainer
6. Update `./.specify/docs/stack.md` with Active Agents list
7. Update `./.specify/docs/index.md` — stack section + `## Engineering Agents`
8. Update root `CLAUDE.md` — point to `./.specify/docs/index.md`, governance, and `README.md`
9. Insert or refresh the managed `README.md` block — overview, getting started, main commands, structure, docs links
10. Update `./.specify/docs/integrations.md` — add detected Axios instances
11. Report: stack detected, 4 reference files loaded, 5 agents configured, docs + entrypoints updated with manual README content preserved
</actions>
</example>
<example>
<user.prompt>Init docs for a new Go project</user.prompt>
<context>Go backend with go.mod. No ./.specify/docs directory.</context>
<actions>
1. Read `go.mod` → detects Go modules and module path
2. Read `references/stack-signals.md` → confirm Go stack
3. Write `./.specify/docs/stack.md` with Go runtime
4. Read `references/references-map.md` → load go.md, hexagonal-architecture.md, error-handling.md, documentation-derivation.md
5. Read `references/agent-map.md` → map backend-architect, reviewer, debugger, task-planner, docs-maintainer
6. Read `references/templates.md` → get template-to-target mapping
7. Read each template, fill with real data, write to `./.specify/docs/`
8. Create or normalize root `CLAUDE.md` as a minimal bridge to the derived docs
9. Create `README.md` or insert the managed `docs-sync` block if the file already exists
10. Create `./.specify/docs/decisions/` directory
11. Report: Go stack detected, 4 reference files loaded, 5 agents configured, docs + entrypoints created with no duplicated architecture prose
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
5. Read `references/references-map.md` → load turborepo.md, monorepo.md, typescript.md, hexagonal-architecture.md
6. Read `references/agent-map.md` → map all relevant agents
7. Update root `./.specify/docs/index.md` and `./.specify/docs/architecture.md` with monorepo-level data
8. Update root `CLAUDE.md` and the managed root `README.md` block from the root docs
9. For apps/web: detect Next.js, write `apps/web/.specify/docs/stack.md` and `apps/web/.specify/docs/index.md`, then create `apps/web/CLAUDE.md` because local docs exist
10. For apps/api: detect NestJS + Prisma, write `apps/api/.specify/docs/stack.md` and `apps/api/.specify/docs/index.md`, then create `apps/api/CLAUDE.md`
11. For packages/ui: detect React, write `packages/ui/.specify/docs/stack.md`; create `packages/ui/CLAUDE.md` only if local `./.specify/docs/` exists
12. Verify `.specify/specs/` exists only at root and that manual README content outside the managed block remains unchanged
13. Report: Monorepo mode, root entrypoints synced, eligible package bridges created, and 0 `specs/` directories created inside packages
</actions>
</example>
</examples>
