---
name: agent-link
description: 'Audit or update `Skills Used` and `Knowledge to Prioritize` pointers in agents so runtime adapters stay aligned with the shared core.'
argument-hint: '[agent name or scope: one/all]'
user-invocable: true
---

# Agent Link

GitHub Copilot adapter for [`../../../skills/agent-link/SKILL.md`](../../../skills/agent-link/SKILL.md).

## When to use

- A new skill or knowledge file changed agent responsibilities
- Copilot and Claude adapters need to stay aligned
- You want to review what a specific agent composes

## Procedure

1. Inspect the target agent or all agents.
2. Compare its skill and knowledge pointers with the current shared core.
3. Update references while keeping each agent single-purpose.
4. Prefer links to shared knowledge over copied guidance.

## References

- [Source workflow](../../../skills/agent-link/SKILL.md)
- [Legacy agents](../../../agents/)
- [Copilot agents](../../../.github/agents/)
