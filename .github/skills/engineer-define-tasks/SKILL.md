---
name: engineer-define-tasks
description: 'Break a spec into small implementation tasks with phases, dependencies, and acceptance criteria. Use when planning delivery before coding.'
argument-hint: '[feature slug or path to spec.md]'
user-invocable: true
---

# Engineer Define Tasks

GitHub Copilot adapter for [`../../../skills/engineer-define-tasks/SKILL.md`](../../../skills/engineer-define-tasks/SKILL.md).

## When to use

- A spec already exists and needs an execution plan
- Work must be split into phases before implementation
- The team needs a clean dependency-aware task list

## Procedure

1. Read the target `spec.md` and relevant project context.
2. Break work into independently completable tasks.
3. Add explicit acceptance criteria and dependencies.
4. Order work as `domain -> use cases -> adapters` and save `tasks.md`.

## References

- [Source workflow](../../../skills/engineer-define-tasks/SKILL.md)
- [Task breakdown knowledge](../../../knowledge/practices/task-breakdown.md)
- [Hexagonal architecture knowledge](../../../knowledge/practices/hexagonal-architecture.md)
