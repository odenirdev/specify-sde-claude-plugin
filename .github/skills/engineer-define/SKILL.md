---
name: engineer-define
description: 'Run the full define phase from `prd.md` to `spec.md` and `tasks.md`. Use when you want a complete technical definition and implementation plan in one step.'
argument-hint: '[feature slug or path to prd.md]'
user-invocable: true
---

# Engineer Define

GitHub Copilot adapter for [`../../../skills/engineer-define/SKILL.md`](../../../skills/engineer-define/SKILL.md).

## When to use

- Run the full define phase end-to-end
- Turn a PRD into both a technical spec and a task breakdown
- Prepare implementation work before delivery starts

## Procedure

1. Locate `prd.md` for the target feature.
2. Run the `engineer-define-spec` workflow to create `spec.md`.
3. Run the `engineer-define-tasks` workflow to create `tasks.md`.
4. Report the generated paths and recommend the next delivery step.

## References

- [Source workflow](../../../skills/engineer-define/SKILL.md)
- [Hexagonal architecture reference](../../../references/practices/hexagonal-architecture.md)
- [Task breakdown reference](../../../references/practices/task-breakdown.md)
