---
name: review-triage
description: 'Analyze a review topic (PR comment, code review feedback) for root cause, impact, and safe fix proposal. Waits for confirmation before implementing.'
argument-hint: '[review comment, PR feedback, or issue description]'
user-invocable: true
---

# Review Triage

GitHub Copilot adapter for [`../../../skills/review-triage/SKILL.md`](../../../skills/review-triage/SKILL.md).

## When to use

- You have a PR comment or code review finding to analyze
- You want cause + impact + a safe fix before touching any code
- You want to generate a structured PR comment to post manually

## Procedure

1. Read the review topic and any referenced file/line in full context.
2. Identify root cause, impact (severity + scope), and ONE safe proposed fix.
3. Present the structured analysis — do NOT apply any change yet.
4. Wait for explicit user confirmation (`yes`) before implementing.

## References

- [Source workflow](../../../skills/review-triage/SKILL.md)
- [Error handling reference](../../../references/utilities/error-handling.md)
- [Testing reference](../../../references/utilities/testing.md)
- [Hexagonal architecture reference](../../../references/practices/hexagonal-architecture.md)
