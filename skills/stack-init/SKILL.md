---
name: stack-init
description: Initialize `.specify/stack.yml` for a project. Creates a minimal declarative YAML as the stack source of truth consumed by stack-enable, stack-disable, and stack-list.
argument-hint: "[--force] [--scope=global | --global]"
---

# Stack Init

## Objective

Create `.specify/stack.yml` as the declarative stack configuration for the current project. This file becomes the primary source of truth for all AI artifact management — which references, skills, and agents are active or disabled.

## When to Use

Use this skill when you need to:
- bootstrap a new project with a `stack.yml` configuration;
- reset the stack configuration after a major restructuring;
- create the file that `stack-enable`, `stack-disable`, and `stack-list` will manage.

## Inputs

Optional flags:
- `--force` — overwrite the target file without asking for confirmation
- `--scope=global` / `--global` — activate all references available in the plugin, skipping project-stack matching

## Target Resolution

The target file is always `.specify/stack.yml`, regardless of whether `plugin.json` exists.

Resolve the target **before** checking for existing configuration. All subsequent steps use this resolved path.

## Responsibilities

### 0. Resolve target file

Set `TARGET = .specify/stack.yml`.

Check for `plugin.json` at the project root:
- Present → set `PLUGIN_CONTEXT = true`
- Absent → set `PLUGIN_CONTEXT = false`

`PLUGIN_CONTEXT` is an auxiliary signal used only in step 2.5. It does not change the target or skip any steps.

### 1. Check for existing configuration

- If `{TARGET}` already exists and `--force` was not provided:
  - Warn the user that the file exists
  - Ask for explicit confirmation before overwriting
  - If the user declines, abort and report no-op

### 2. Discover project context

Before writing, scan the project to infer which artifacts are relevant:

**2.1 — Enumerate available artifacts**

Collect every artifact the plugin exposes:
- References: glob `references/**/*.md` (deduplicate nested duplicates — keep the shortest canonical path)
- Skills: glob `skills/*/SKILL.md` → extract the `name` from frontmatter
- Agents: glob `agents/*.md` → extract the `name` from frontmatter (if any)

**2.2 — Detect project stack**

Read the following signals from the project root and subdirectories:
- `package.json` / `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` → JavaScript/TypeScript dependencies
- `go.mod` → Go modules
- `requirements.txt` / `pyproject.toml` → Python packages
- `*.config.ts` / `*.config.js` (vite, jest, tailwind, etc.) → tooling
- `Dockerfile` / `serverless.yml` / `serverless.ts` → platform signals
- `.specify/docs/stack.md` → pre-detected stack summary (highest priority if present)
- Top-level directory names (`apps/`, `packages/`, `services/`) → monorepo structure

**2.3 — Match artifacts to project**

If `--scope=global` was provided, skip matching entirely: include **all** references discovered in step 2.1 in `active`. Skip to step 2.4.

Otherwise, for each available reference, apply the matching rules below.
Include matched references in `active`. Unmatched references are omitted — absence implies inactive.

| Reference path contains | Match signal |
|---|---|
| `react` | `react` in deps or imports |
| `nestjs` | `@nestjs/core` in deps |
| `langgraph` | `langgraph` or `@langchain` in deps |
| `serverless-framework` | `serverless.yml` or `serverless` in deps |
| `ionic-react` | `@ionic/react` in deps |
| `capacitor` | `@capacitor/core` in deps |
| `prisma` | `prisma` or `@prisma/client` in deps |
| `vite` | `vite` in deps or config files |
| `turborepo` | `turbo` in deps or `turbo.json` present |
| `axios` | `axios` in deps |
| `zod` | `zod` in deps |
| `formik` | `formik` in deps |
| `tanstack-query` | `@tanstack/react-query` in deps |
| `react-router-dom` | `react-router-dom` in deps |
| `langchain-core` | `@langchain/core` in deps |
| `langchain-groq` | `@langchain/groq` in deps |
| `ionicons` | `ionicons` in deps |
| `typescript` | `typescript` in deps or `.ts` files present |
| `go` | `go.mod` present |
| `aws-lambda` | `aws-lambda` in deps or Dockerfile targets Lambda |
| `hexagonal-architecture` | `src/domain/` or `src/application/` directories present |
| `monorepo` | `packages/`, `apps/`, or `turbo.json` present |
| `api-design` | `src/routes/`, `src/controllers/`, or `src/handlers/` present |
| `task-breakdown` | always active — universally applicable |
| `documentation-derivation` | always active — universally applicable |
| `testing` | test files (`*.test.*`, `*.spec.*`) or test config present |
| `error-handling` | always active — universally applicable |
| `logging` | always active — universally applicable |

**2.4 — Classify skills**

- Stack management skills (`stack-init`, `stack-enable`, `stack-disable`, `stack-list`) → always `active`
- All other discovered skills → `active` by default
- Skills with no matching SKILL.md file → omit entirely

**2.5 — Classify agents**

Apply the matching rules below. Include matched agents in `active`. Unmatched agents are omitted.

| Agent name | Match signal |
|---|---|
| `reviewer` | always active — universally applicable |
| `debugger` | always active — universally applicable |
| `docs-maintainer` | always active — universally applicable |
| `task-planner` | always active — universally applicable |
| `backend-architect` | `@nestjs/core` in deps |
| `langgraph-architect` | `langgraph` or `@langchain/core` in deps |

If `PLUGIN_CONTEXT = true`: all discovered agents → `active` (the project is the plugin itself, so all agents are relevant by definition).

If no agents directory exists → set `agents.active: []`.

### 3. Write `{TARGET}`

Write the file using the discovered context. Use the canonical format below.

### 4. Confirm and report

After writing the file, output a confirmation using the output format below.

## Canonical YAML Format

```yaml
version: "1"

references:
  active:
    - references/practices/hexagonal-architecture.md
    - { id: references/frameworks/react.md, scope: packages/app }

skills:
  active:
    - stack-list
    - stack-enable
    - stack-disable
    - stack-init

agents:
  active:
    - reviewer
    - backend-architect
```

**Format rules:**
- Simple entries: plain string with the canonical identifier
- Scoped entries (monorepo): inline object `{ id: "...", scope: "packages/app" }`
- Empty sections must use `[]`, not be omitted
- `version: "1"` is required at the top level
- No `disabled` key — absence from `active` implies inactive

## Output Format

```md
Result: created | no-op
Target: `{TARGET}`

## References
Active: N

## Skills
Active: N

## Agents
Active: N
```

## Guardrails

- Never write outside `{TARGET}` without explicit user confirmation
- Follow the canonical YAML format exactly; do not add extra keys or sections

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
