---
name: engineer-define-tasks
description: Breaks a feature definition or spec into small, executable engineering tasks with phases, acceptance criteria, and dependencies. Triggered when the user wants to plan implementation tasks, create a task list from a spec, or define work units before coding begins.
argument-hint: "[feature name, spec path, or problem description]"
allowed-tools: Read, Glob, Grep, Write
---

# Engineering Tasks

## Objective

Produce a structured task breakdown that converts a technical definition or spec into an executable implementation plan. Tasks must be small enough to complete in a focused session, testable in isolation, and ordered by dependency.

## When to Use

Use this skill when the user wants to:
- Break a feature into implementation tasks
- Convert a spec from `./specify/specs` into actionable work
- Create a phased plan before writing code
- Define acceptance criteria for each unit of work
- Identify task dependencies and unblock parallel work

## Inputs

- Technical definition (from `engineer-define-spec`) or spec path
- (Optional) Existing code context for accurate task scoping
- (Optional) Team constraints or delivery phases

## Responsibilities

### 1. Read the Spec or Definition First
Before breaking down tasks:
- Read the spec from `./specify/specs` if it exists
- Read relevant domain models, use cases, and tests
- Understand what already exists — don't create tasks for things that are already done

### 2. Identify Phases
Group work into sequential phases where each phase produces something demonstrably working:
```
Phase 1 — Domain Model
Phase 2 — Application Logic (Use Cases)
Phase 3 — Infrastructure (Adapters, DB)
Phase 4 — API Layer
Phase 5 — Tests & Documentation
```
Adapt phases to the actual work — don't force the template.

### 3. Write Tasks
For each task:
- Use a verb + noun format: "Implement CreateOrder use case"
- One clear objective per task
- Completable in hours, not days
- Has a test that verifies it works

### 4. Write Acceptance Criteria
Each task must have explicit acceptance criteria:
- Behavioral assertions, not implementation steps
- Testable: "Given X, when Y, then Z"
- Cover: happy path, failure paths, edge cases

### 5. Map Dependencies
Identify which tasks block others. List dependencies explicitly:
```
T4 depends on: T1 (domain model must exist before use case)
T7 depends on: T2 (interface must exist before implementation)
```

## Output Format

```
## Task Breakdown: [Feature Name]

### Phase 1 — [Phase Name]
**Goal**: [What this phase delivers]

- [ ] **T1** — [Task name]
  - Objective: [One sentence]
  - AC:
    - [ ] [Behavioral assertion]
    - [ ] [Error case]
  - Dependencies: none

- [ ] **T2** — [Task name]
  - Objective: [One sentence]
  - AC:
    - [ ] [Behavioral assertion]
  - Dependencies: T1

### Phase 2 — [Phase Name]
...

### Dependency Map
T1 → T2 → T4
T3 → T5

### Notes
[Any context, constraints, or decisions that affect execution]
```

## Output Location

Write the output to `./specify/specs/[feature-slug]/tasks.md`. If the directory does not exist, create it.

The file must start with:
```
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---
```

If the file already exists, update its content and refresh `updated_at`. Do not append — replace in place.

## Quality Bar

A task breakdown is complete when:
- Every task has at least one acceptance criterion
- No task requires another undocumented task to complete first
- Each phase delivers something verifiable
- Tasks are small enough that "done" is unambiguous

## Knowledge to Consult

- `knowledge/practices/task-breakdown.md` — task breakdown patterns
- `knowledge/practices/hexagonal-architecture.md` — for ordering domain → app → infra

## Repository Areas to Inspect

- `./specify/specs` — source spec or definition
- Existing domain entities to avoid recreating what's there
- Test files to understand current coverage baseline

## Guardrails

- Do not create tasks for code that already exists
- Do not merge multiple concerns into one task to save time
- Do not skip the test task — it is a first-class deliverable
- Do not assume the spec is complete — list gaps as open items

## Example

User: "Break down the payment integration feature"

Actions:
1. Read `./specify/specs/payment-integration` if it exists
2. Grep for existing payment-related code
3. Identify what exists (domain model? repository?) vs what needs to be built
4. Phase: Domain Model → Use Cases → Stripe Adapter → API → Tests
5. Write tasks with ACs for each phase
6. Save to `./specify/specs/payment-integration/tasks.md`
