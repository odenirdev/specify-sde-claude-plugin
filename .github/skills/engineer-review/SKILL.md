---
name: engineer-review
description: 'Review code, diffs, or modules for bugs, risks, architecture alignment, and production concerns. Use before merge or deploy.'
argument-hint: '[path, diff, module, or PR context]'
user-invocable: true
---

# Engineer Review

GitHub Copilot adapter for [`../../../skills/engineer-review/SKILL.md`](../../../skills/engineer-review/SKILL.md).

## When to use

- Reviewing a PR or local diff
- Auditing a module for correctness and risk
- Checking if a change is safe to ship

## Procedure

1. Read the diff or target files fully before commenting.
2. Compare the change against specs, tests, and shared architecture rules.
3. Report only issues supported by evidence.
4. Separate critical findings from improvements and strengths.

## References

- [Source workflow](../../../skills/engineer-review/SKILL.md)
- [Error handling reference](../../../references/utilities/error-handling.md)
- [Testing reference](../../../references/utilities/testing.md)
- [Hexagonal architecture reference](../../../references/practices/hexagonal-architecture.md)
