---
name: engineer-delivery
description: 'Execute the delivery phase from `tasks.md`, track progress, and dispatch work in dependency order. Use when implementation is ready to begin.'
argument-hint: '[feature slug or path to tasks.md]'
user-invocable: true
---

# Engineer Delivery

GitHub Copilot adapter for [`../../../skills/engineer-delivery/SKILL.md`](../../../skills/engineer-delivery/SKILL.md).

## When to use

- The task plan is approved and coding should start
- Work needs to be executed in phases with verification at each step
- A delivery report or implementation checkpoint is needed

## Procedure

1. Read the target `tasks.md` and confirm dependency order.
2. Execute one task at a time, verifying each outcome before moving on.
3. Prefer existing specialists and shared knowledge instead of ad-hoc instructions.
4. Record completion or remaining blockers in the delivery output.

## References

- [Source workflow](../../../skills/engineer-delivery/SKILL.md)
- [Task breakdown knowledge](../../../knowledge/practices/task-breakdown.md)
- [Testing knowledge](../../../knowledge/utilities/testing.md)
