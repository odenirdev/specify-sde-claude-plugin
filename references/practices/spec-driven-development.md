# Spec-Driven Development — Engineering Reference

## Core Principle

A spec is the source of truth for a feature. It exists before any code, any task, and any agent prompt. The quality of what the AI produces directly correlates with the precision of the spec that drives it. Vague specs generate vague code — and vague code generates debugging sessions.

## Workflow

Every feature follows four stages:

```
Specify → Plan → Tasks → Implement
```

| Stage | Output | Who owns it |
|---|---|---|
| **Specify** | `spec.md` — problem, approach, constraints | Human + AI (collaborative) |
| **Plan** | Architecture decisions, component map, trade-offs | AI (reviewed by human) |
| **Tasks** | Numbered task list with acceptance criteria | AI (reviewed by human) |
| **Implement** | Code, tests, docs — one task at a time | AI (per-task) |

Human approval gates each stage transition. The AI never moves forward without confirmation.

---

## Spec Structure

A spec lives at `.specify/specs/<slug>/spec.md`. Minimal required sections:

```markdown
# Spec: Feature Name

## Problem
One paragraph. What is broken or missing? Who is affected? What is the current behavior?

## Approach
How will this be solved? Architecture decision, not implementation detail.
State what is in scope and explicitly what is out of scope.

## Constraints
- Must not break existing behavior X
- Must be compatible with runtime Y
- Must not add dependency Z without justification

## Acceptance Criteria
- [ ] Behavior A works under condition B
- [ ] Error case C is handled with response D
- [ ] No regression in E

## Tasks
Generated after human approval of the approach. See task-breakdown reference.
```

---

## Spec Quality Rules

### Problem Statement
- Describe the gap, not the solution
- Include who is affected: "users cannot", "the agent fails to", "deploying to X is impossible without"
- State current behavior vs. desired behavior explicitly

### Approach
- One chosen direction — not a menu of options
- If multiple approaches were considered, record the rejected ones and why in a decision log
- Reference architectural constraints: "must fit the shared-core / adapter pattern"

### Constraints
- Negative constraints are as important as positive ones: "must NOT touch Y"
- Constraints come from architecture decisions, existing contracts, and team agreements
- Every constraint must be verifiable — if you can't check it, it's not a constraint, it's a wish

### Acceptance Criteria
- Written in behavior terms, not implementation terms
- Each criterion is independently verifiable
- Include at least one error/failure path

---

## Task Generation from Spec

Once the spec is approved, generate tasks following the task-breakdown reference:

1. Extract each acceptance criterion as a candidate task
2. Group by dependency order (foundation → core logic → integration → tests)
3. Assign acceptance criteria to each task
4. Mark inter-task dependencies explicitly
5. Each task must be completable in isolation — if it can't, split it

---

## AI Agent Integration

When working with AI agents in a spec-driven workflow:

- **Always provide the spec** — do not ask the agent to infer the goal from context
- **One task per agent invocation** — scope each prompt to a single task
- **Include current task + dependencies** — the agent needs to know what already exists
- **Review before proceeding** — human confirms each task output before the next task starts
- **Spec wins over agent suggestions** — if the agent proposes something outside the spec, redirect

```
Bad:  "Implement the user authentication feature."
Good: "Implement Task T3: Create JWT token generation service.
       Spec: .specify/specs/user-auth/spec.md
       Dependencies: T1 (User entity), T2 (UserRepository interface) — both complete.
       AC: token expires in 1h, contains userId and role, is signed with RS256."
```

---

## Spec Lifecycle

```
draft → approved → in-progress → done → archived
```

- **draft**: being written, not yet reviewed
- **approved**: human has confirmed approach and constraints
- **in-progress**: tasks are being implemented
- **done**: all acceptance criteria verified
- **archived**: superseded by a newer spec or decision

Do not delete specs — move them to `archived/`. They are decision history.

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Spec written after implementation | No source of truth; post-hoc rationalization |
| Approach section lists options without choosing | Agent must guess; output is inconsistent |
| No acceptance criteria | Done is undefined; review is subjective |
| Tasks generated before spec is approved | Rework if approach changes |
| Vague problem ("improve performance") | Unmeasurable, un-scoped |
| One spec for multiple unrelated features | Conflates scope; hard to verify independently |

---

## Review Criteria

- [ ] Problem describes gap, not solution
- [ ] Approach states one chosen direction
- [ ] Out-of-scope is explicit
- [ ] Constraints are verifiable
- [ ] Acceptance criteria cover at least one error path
- [ ] Tasks derive from acceptance criteria
- [ ] Each task is independently completable and testable
- [ ] Spec lives at `.specify/specs/<slug>/spec.md`
