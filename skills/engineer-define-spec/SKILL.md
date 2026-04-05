---
name: engineer-define-spec
description: Transforms a problem statement or feature request into a structured technical definition — approach, scope, architecture impact, and risks. Triggered when the user wants to define a solution, break down a feature technically, or frame a problem before implementation.
argument-hint: "[problem description or feature request]"
allowed-tools: Read, Glob, Grep
---

# Engineering Define

## Objective

Transform an ambiguous problem or feature request into a concrete technical definition that a developer can act on. The output is a structured definition document — not a full implementation plan, but a clear framing of what needs to be built, how, and where the risks are.

## When to Use

Use this skill when the user wants to:
- Define the technical approach for a new feature
- Frame a problem before planning tasks
- Decide between implementation strategies
- Identify scope and constraints before writing code
- Produce an engineering spec from a product requirement

## Inputs

- Problem description, feature request, or user story
- (Optional) Existing specs in `./.specify/specs`
- (Optional) Architectural constraints or system context

## Responsibilities

### 1. Read the Context First
Before defining anything:
- Check `./.specify/specs` for any related specs
- Grep the codebase for related modules, entities, and patterns
- Identify existing conventions to follow or constraints to respect

### 2. Frame the Problem Precisely
Restate the problem as an engineering statement:
- What is the system being asked to do?
- What are the boundaries? (what is in scope and explicitly out of scope)
- What already exists that this touches or depends on?

### 3. Define the Approach
Describe the technical solution at an architectural level:
- What components will be created or modified?
- What is the data flow?
- What persistence, APIs, or integrations are involved?
- What patterns and conventions will be followed?

Do not write code here — describe what will be built.

### 4. Identify Architecture Impact
- Does this introduce new domain concepts?
- Does it require schema changes?
- Does it touch any cross-cutting concerns (auth, observability, caching)?
- Does it change any existing interfaces or contracts?

### 5. Surface Risks and Open Questions
Be honest about:
- What is not well understood yet
- Where assumptions are being made
- Dependencies on external systems or teams
- Performance or scalability concerns
- Security implications

## Output Format

```
## Technical Definition: [Feature/Problem Name]

### Problem
[Precise engineering statement of what needs to be solved]

### Scope
**In scope**: [what will be built]
**Out of scope**: [what is explicitly excluded]

### Approach
[Architectural description of the solution — components, data flow, patterns]

### Architecture Impact
[What changes, what new concepts are introduced, what contracts are affected]

### Risks
[Ranked list of risks with brief description]

### Open Questions
[Questions that must be answered before implementation begins]

### Next Step
[What to do immediately after this definition is reviewed — usually: write spec or start task breakdown]
```

## Output Location

Write the output to `./.specify/specs/[feature-slug]/definition.md`. If the directory does not exist, create it.

The file must start with:
```
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---
```

If the file already exists, update its content and refresh `updated_at`. Do not append — replace in place.

## Quality Bar

A definition is complete when:
- The scope is unambiguous — no reasonable person would interpret it differently
- The approach can be implemented by a developer without further ambiguity on the core path
- All known risks are surfaced
- Open questions are listed, not assumed away

## References to Consult

- `references/practices/hexagonal-architecture.md` — architecture decisions and layer boundaries
- `references/practices/api-design.md` — API contract decisions and REST conventions
- `references/practices/task-breakdown.md` — scoping tasks after definition
- `references/utilities/error-handling.md` — error propagation and failure modes
- `references/languages/typescript.md` — TypeScript patterns (load if detected in stack)
- `references/languages/go.md` — Go patterns (load if detected in stack)
- `references/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `references/frameworks/react.md` — React patterns (load if detected in stack)
- `references/frameworks/ionic-react.md` — Ionic React component and routing patterns (load if detected in stack)
- `references/frameworks/capacitor.md` — Capacitor native integration patterns (load if detected in stack)
- `references/utilities/testing.md` — test strategy and acceptance criteria in spec
- `references/libs/prisma.md` — Prisma/ORM patterns (load if detected in stack)
- `references/libs/axios.md` — HTTP client patterns (load if detected in stack)
- `references/libs/react-router-dom.md` — React Router v5 routing patterns (load if detected in stack)
- `references/libs/vite.md` — Vite build tool configuration (load if detected in stack)
- `references/libs/ionicons.md` — Ionicons integration constraints (load if detected in stack)

## Repository Areas to Inspect

- `./.specify/specs` — existing specs for related features
- Domain entities and use cases related to the feature
- Schema/migrations if persistence is involved
- API contracts if integration is involved

## Guardrails

- Do not start implementing — this is a definition phase
- Do not resolve open questions unilaterally — list them and ask
- Do not over-engineer — define the minimal effective solution
- Do not assume existing code quality — read before referencing

## Example

User: "I need to add multi-tenancy to the existing user module"

Actions:
1. Read `./.specify/specs` for user-related specs
2. Grep for User entity, UserRepository, and UserController
3. Identify what changes to the schema, queries, and auth are required
4. Frame the scope: tenant isolation model (row-level vs schema-per-tenant)
5. Identify risks: data leakage, migration complexity, performance impact
6. List open questions: tenant provisioning flow, cross-tenant admin access
7. Produce definition document
