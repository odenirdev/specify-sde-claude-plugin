---
name: engineer-discovery
description: 'Capture feature requirements through a structured discovery interview and write `prd.md`. Use when starting a new feature or clarifying scope before technical design.'
argument-hint: '[feature name or problem description]'
user-invocable: true
---

# Engineer Discovery

GitHub Copilot adapter for the reusable workflow in [`../../../skills/engineer-discovery/SKILL.md`](../../../skills/engineer-discovery/SKILL.md).

## When to use

- Start a new feature from a rough idea
- Capture requirements before generating a spec
- Produce a structured `prd.md` from a problem statement

## Procedure

1. Check `./.specify/specs/` and `./.specify/docs/` for overlapping work or constraints.
2. Interview the user about problem, goals, users, requirements, non-functional constraints, out-of-scope items, and acceptance criteria.
3. Derive a kebab-case feature slug.
4. Write `./.specify/specs/<slug>/prd.md` with explicit, testable requirements.

## References

- [Source workflow](../../../skills/engineer-discovery/SKILL.md)
- [Documentation derivation reference](../../../references/practices/documentation-derivation.md)
- [Task breakdown reference](../../../references/practices/task-breakdown.md)
