---
name: engineer-debug
description: 'Perform root cause analysis for bugs, failing tests, regressions, or incidents. Use when you need structured hypotheses and evidence to collect.'
argument-hint: '[bug description, failing test, or incident summary]'
user-invocable: true
---

# Engineer Debug

GitHub Copilot adapter for [`../../../skills/engineer-debug/SKILL.md`](../../../skills/engineer-debug/SKILL.md).

## When to use

- A failure is real but the cause is unclear
- A regression appeared after a recent change
- A test or integration behaves unexpectedly

## Procedure

1. Reproduce and describe the symptom precisely.
2. Trace the relevant execution path and recent changes.
3. Form hypotheses with confirming or rejecting evidence.
4. Propose the smallest investigation steps needed to reach root cause.

## References

- [Source workflow](../../../skills/engineer-debug/SKILL.md)
- [Error handling reference](../../../references/utilities/error-handling.md)
- [Logging reference](../../../references/utilities/logging.md)
