---
name: engineer-delivery
description: Executes the task plan from tasks.md by dispatching work to specialized subagents and tracking completion. Triggered when the user wants to begin implementation, execute the task plan, start delivery for a feature, or run the delivery phase of the engineering pipeline.
argument-hint: "[feature slug or path to tasks.md]"
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Agent
---

# Engineer Delivery

## Objective

Read `tasks.md` for a feature, dispatch each task to the appropriate specialized subagent, track progress phase by phase, and produce a delivery report. The user should be able to start implementation from a single command.

## When to Use

Use this skill when the user wants to:
- Begin implementing a feature that has a completed `tasks.md`
- Execute the full task plan via subagents
- Run the delivery phase of the engineering pipeline
- Automate the handoff from planning to implementation

## Inputs

- Feature slug (e.g., `user-auth`) or direct path to `tasks.md`
- `./specify/specs/[slug]/tasks.md` must already exist
- `./specify/specs/[slug]/spec.md` should exist as reference

## Responsibilities

### 1. Locate the Task Plan

- If the user provides a slug, read `./specify/specs/[slug]/tasks.md`
- If the slug is not provided, check `./specify/specs/` for the most recent spec with a `tasks.md`
- If `tasks.md` is not found, stop and instruct the user to run `engineer-define` first

### 2. Parse Phases and Tasks

Read `tasks.md` and extract:
- All phases with their descriptions
- All tasks within each phase, including:
  - Task title and description
  - Acceptance criteria
  - Dependencies (which tasks must complete first)
  - Task type: `backend`, `frontend`, `test`, `docs`, `infra`, or `general`

### 3. Dispatch Tasks by Phase

Execute phases sequentially. Within each phase, tasks without dependencies can be dispatched in parallel:

| Task Type | Subagent to Use |
|-----------|----------------|
| `backend` | `specify-sde:backend-architect` |
| `frontend` | Dispatch to a React or frontend-focused agent |
| `test` | `specify-sde:test-engineer` (if available) or implement inline |
| `docs` | `specify-sde:docs-maintainer` |
| `infra` | Implement inline with Bash tools |
| `general` | Implement inline |

For each task dispatched to a subagent:
- Provide: task description, acceptance criteria, relevant file paths from codebase
- Include: path to `spec.md` and `tasks.md` as context
- Wait for confirmation before marking task complete

### 4. Track Progress

After each task completes:
- Mark the task as done in the delivery report (not in `tasks.md`)
- Verify acceptance criteria are met before proceeding to the next dependent task
- If a task fails or is blocked, surface the blocker to the user before continuing

### 5. Write Delivery Report

After all phases complete, write `./specify/specs/[slug]/delivery.md` with a summary of what was implemented.

## Output Format

Inline progress updates during execution:

```
## Phase 1: [Phase Name]
- [x] Task 1: [title] → done
- [x] Task 2: [title] → done
- [ ] Task 3: [title] → in progress

## Phase 2: [Phase Name]
...
```

Delivery report written to `delivery.md`:

```markdown
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---

# Delivery Report: [Feature Name]

## Summary

[1-2 sentences on what was built]

## Phases Completed

### Phase 1: [Name]
- [Task title] — [brief outcome]

### Phase 2: [Name]
- [Task title] — [brief outcome]

## Files Modified

- [path/to/file] — [what changed]

## Acceptance Criteria Status

- [x] [Criterion 1] — verified
- [ ] [Criterion 2] — pending manual verification

## Known Issues / Deferred Items

- [Any items not completed or deferred to next iteration]

## Next Step

Run `engineer-iteration` to review and refine the delivery.
```

## Output Location

- Progress: inline during execution
- Delivery report: `./specify/specs/[slug]/delivery.md`

## Quality Bar

Delivery is complete when:
- All tasks in `tasks.md` have been attempted
- Each task's acceptance criteria has been checked
- The delivery report lists files modified and criteria status
- No task is silently skipped — deferred items are explicitly listed

## Guardrails

- Do not start delivery if `tasks.md` is missing — run `engineer-define` first
- Do not skip phases — execute in order, respecting dependencies
- Do not mark a task complete if its acceptance criteria are not met
- Do not swallow errors from subagents — surface blockers immediately
- Do not modify `tasks.md` or `spec.md` during delivery

## Example

User: "run engineer-delivery for user-notifications"

Actions:
1. Read `./specify/specs/user-notifications/tasks.md` — 3 phases, 9 tasks
2. Phase 1 (Core backend):
   - Dispatch task "Create Notification domain entity" to `backend-architect`
   - Dispatch task "Create NotificationService use case" to `backend-architect`
   - Wait for both; verify acceptance criteria
3. Phase 2 (API layer):
   - Dispatch task "Add POST /notifications endpoint" to `backend-architect`
   - Wait; verify response format matches spec
4. Phase 3 (Tests):
   - Dispatch "Write unit tests for NotificationService" inline
   - Verify tests pass with `Bash`
5. Write `./specify/specs/user-notifications/delivery.md`
6. Report: "3 phases complete. 9/9 tasks done. Next: engineer-iteration"
