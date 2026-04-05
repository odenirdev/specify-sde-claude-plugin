---
name: engineer-define-spec
description: 'Transform a problem statement or PRD into a technical spec with scope, approach, architecture impact, risks, and acceptance criteria. Use before implementation planning.'
argument-hint: '[feature slug, request, or path to prd.md]'
user-invocable: true
---

# Engineer Define Spec

GitHub Copilot adapter for [`../../../skills/engineer-define-spec/SKILL.md`](../../../skills/engineer-define-spec/SKILL.md).

## When to use

- A feature needs a technical definition
- A rough request should be turned into a concrete solution shape
- Architecture impact and risks should be clarified before delivery

## Procedure

1. Read the PRD or problem statement.
2. Translate requirements into scope, constraints, architecture impact, and acceptance criteria.
3. Keep the solution aligned with current repository patterns and shared knowledge.
4. Write `./.specify/specs/<slug>/spec.md`.

## References

- [Source workflow](../../../skills/engineer-define-spec/SKILL.md)
- [Hexagonal architecture knowledge](../../../knowledge/practices/hexagonal-architecture.md)
- [API design knowledge](../../../knowledge/practices/api-design.md)
- [Error handling knowledge](../../../knowledge/utilities/error-handling.md)
