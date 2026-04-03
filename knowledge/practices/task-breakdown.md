# Task Breakdown — Engineering Reference

## Core Principle

A well-broken task is completable in isolation, testable, and reviewable. A task that requires five other things to be done first, or cannot be verified without running the whole system, is not a task — it's a milestone.

## Breakdown Principles

### The Right Grain
A good task:
- Has one clear objective
- Can be completed in a single work session (hours, not days)
- Has explicit, verifiable acceptance criteria
- Has no hidden dependencies that aren't listed
- Can be tested in isolation

### Phased Breakdown
Group tasks into phases that deliver incremental value:

```
Phase 1 — Foundation
  [T1] Define domain model and types
  [T2] Write repository interface
  [T3] Implement in-memory fake repository for tests

Phase 2 — Core Logic
  [T4] Implement CreateUser use case
  [T5] Implement FindUser use case
  [T6] Implement UpdateUser use case

Phase 3 — Infrastructure
  [T7] Implement Prisma repository adapter
  [T8] Wire up dependency injection in module
  [T9] Add database migration

Phase 4 — API Layer
  [T10] Create DTOs with validation
  [T11] Implement controller with endpoints
  [T12] Add Swagger documentation

Phase 5 — Tests & Quality
  [T13] Unit tests for use cases
  [T14] Integration tests for repository
  [T15] E2E tests for API endpoints
```

### Acceptance Criteria Format
Each task must state what "done" means:
```
Task: Implement CreateUser use case
AC:
- [ ] User is persisted with hashed password
- [ ] Duplicate email throws ConflictError
- [ ] Empty name throws ValidationError
- [ ] Returns created user without password field
- [ ] Unit test covers all 4 cases above
```

---

## Dependency Mapping
Identify blockers before starting:
```
T4 depends on: T1, T2
T7 depends on: T2 (interface must exist before implementation)
T11 depends on: T4, T5, T6
```

Visualize as a DAG when parallelism is important. Unblock parallel tracks where possible.

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| "Implement the whole auth system" | Unestimable, unverifiable |
| Tasks without acceptance criteria | Done is undefined |
| All tasks in one phase | No incremental delivery |
| Hidden dependencies | Blocks others without warning |
| Tasks scoped to files, not behavior | Misses architectural context |

---

## Task Template

```markdown
## [TN] Task name

**Objective**: One sentence describing the outcome.

**Acceptance Criteria**:
- [ ] Behavior 1
- [ ] Behavior 2
- [ ] Error case handled

**Dependencies**: T1, T2

**Notes**: Any relevant context, constraints, or references.
```

---

## Review Criteria

- [ ] Tasks are independently completable
- [ ] Each task has explicit acceptance criteria
- [ ] Dependencies are listed
- [ ] Phase grouping reflects incremental delivery
- [ ] Tasks are testable without full system running
