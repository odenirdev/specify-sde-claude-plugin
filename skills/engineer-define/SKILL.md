---
name: engineer-define
description: Orchestrates the full definition phase by sequentially invoking engineer-define-spec then engineer-define-tasks. Triggered when the user wants to go from a prd.md to a complete spec + task breakdown in one step, run the define phase end-to-end, or generate both spec.md and tasks.md for a feature.
argument-hint: "[feature slug or path to prd.md]"
allowed-tools: Read, Glob, Grep, Write
---

# Engineer Define

## Objective

Orchestrate the complete definition phase for a feature: read `prd.md`, produce `spec.md` via `engineer-define-spec`, then produce `tasks.md` via `engineer-define-tasks`. The user receives both artifacts in a single invocation.

## When to Use

Use this skill when the user wants to:
- Run the full define phase in one step
- Go from a PRD directly to a task breakdown
- Generate `spec.md` and `tasks.md` together without invoking each sub-skill separately

## Inputs

- Feature slug (e.g., `user-auth`) or direct path to `prd.md`
- `./.specify/specs/[slug]/prd.md` must already exist

## Responsibilities

### 1. Locate the PRD

- If the user provides a slug, check `./.specify/specs/[slug]/prd.md`
- If the user provides a path, verify the file exists
- If `prd.md` is not found, stop and instruct the user to run `engineer-discovery` first

### 2. Invoke engineer-define-spec

Run the `engineer-define-spec` skill with the `prd.md` as input:
- Read `prd.md` to extract problem, objectives, functional requirements, and acceptance criteria
- Produce `./.specify/specs/[slug]/spec.md` following the spec output format
- Do not proceed to the next step until `spec.md` is written

### 3. Invoke engineer-define-tasks

Run the `engineer-define-tasks` skill with the `spec.md` as input:
- Read the freshly written `spec.md`
- Break scope into phased, executable tasks with acceptance criteria and dependencies
- Produce `./.specify/specs/[slug]/tasks.md`

### 4. Report Completion

Confirm to the user:
- Path to `spec.md`
- Path to `tasks.md`
- Number of phases and tasks generated
- Next step: `engineer-delivery`

## Output Format

No standalone output artifact. Reports completion inline:

```
## Define Phase Complete: [feature-slug]

- spec.md written → ./.specify/specs/[slug]/spec.md
- tasks.md written → ./.specify/specs/[slug]/tasks.md

[N] phases · [M] tasks total

Next: run engineer-delivery to begin implementation.
```

## Output Location

Delegates to sub-skills:
- `./.specify/specs/[slug]/spec.md` (written by `engineer-define-spec`)
- `./.specify/specs/[slug]/tasks.md` (written by `engineer-define-tasks`)

## Quality Bar

The define phase is complete when:
- `spec.md` exists and covers all PRD requirements with defined scope, approach, and acceptance criteria
- `tasks.md` exists with at least one phase and tasks that map to spec requirements
- No open requirements from `prd.md` are left unaddressed

## References to Consult

- `references/practices/hexagonal-architecture.md` — architecture decisions and layer boundaries
- `references/practices/task-breakdown.md` — task phasing and acceptance criteria
- `references/practices/api-design.md` — API contract decisions and REST conventions
- `references/utilities/error-handling.md` — error propagation and failure modes
- `references/utilities/testing.md` — test strategy and acceptance criteria
- `references/languages/typescript.md` — TypeScript patterns (load if detected in stack)
- `references/languages/go.md` — Go patterns (load if detected in stack)
- `references/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `references/frameworks/react.md` — React patterns (load if detected in stack)
- `references/frameworks/ionic-react.md` — Ionic React component and routing patterns (load if detected in stack)
- `references/frameworks/capacitor.md` — Capacitor native integration patterns (load if detected in stack)
- `references/libs/prisma.md` — Prisma/ORM patterns (load if detected in stack)
- `references/libs/axios.md` — HTTP client patterns (load if detected in stack)
- `references/libs/react-router-dom.md` — React Router v5 routing patterns (load if detected in stack)
- `references/libs/vite.md` — Vite build tool configuration (load if detected in stack)
- `references/libs/ionicons.md` — Ionicons integration constraints (load if detected in stack)

## Guardrails

- Do not skip `engineer-define-spec` — `tasks.md` depends on `spec.md`
- Do not run if `prd.md` is missing — tell the user to run `engineer-discovery` first
- Do not modify `prd.md` — it is read-only input
- Do not invent requirements not present in `prd.md`

## Example

User: "run engineer:define for user-auth"

Actions:
1. Check `./.specify/specs/user-auth/prd.md` — found
2. Run `engineer-define-spec`: read prd.md → write spec.md with scope, approach, constraints, acceptance criteria
3. Run `engineer-define-tasks`: read spec.md → write tasks.md with Phase 1 (core), Phase 2 (edge cases), Phase 3 (tests)
4. Report: "spec.md + tasks.md written. 3 phases, 11 tasks. Next: engineer-delivery"
