---
name: knowledge-link
description: 'Audit or update the `Knowledge to Consult` sections in skills so each workflow points to the right shared knowledge files.'
argument-hint: '[skill name or scope: one/all]'
user-invocable: true
---

# Knowledge Link

GitHub Copilot adapter for [`../../../skills/knowledge-link/SKILL.md`](../../../skills/knowledge-link/SKILL.md).

## When to use

- A new knowledge file was added
- Skill references drifted from the current knowledge base
- You want to audit which knowledge each skill uses

## Procedure

1. Inspect the target skill or the full skill set.
2. Compare current `Knowledge to Consult` pointers with available shared knowledge.
3. Update links to improve reuse and avoid duplication.
4. Keep knowledge references explicit and minimal.

## References

- [Source workflow](../../../skills/knowledge-link/SKILL.md)
- [Knowledge directory](../../../knowledge/)
