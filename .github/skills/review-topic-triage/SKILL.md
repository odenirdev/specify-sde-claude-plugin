---
name: review-topic-triage
description: 'Triage a single PR review finding: analyze cause, assess impact, propose a safe fix, and wait for confirmation before applying any change.'
argument-hint: '[review comment, finding, or file:line reference]'
user-invocable: true
---

# Review Topic Triage

GitHub Copilot adapter for [`../../../skills/review-topic-triage/SKILL.md`](../../../skills/review-topic-triage/SKILL.md).

## When to use

- A PR review comment needs structured analysis before acting
- You want to understand the root cause and blast radius of a finding
- You need a safe fix proposal gated on explicit confirmation

## Procedure

1. Read the flagged file and its callers before forming any opinion.
2. Identify the root cause — is the reviewer correct? Is it pre-existing?
3. Assess impact: severity, scope, and risk of leaving it unfixed.
4. Propose the minimal safe fix with trade-offs named.
5. **Stop. Wait for user confirmation before writing any code.**

## References

- [Source workflow](../../../skills/review-topic-triage/SKILL.md)
- [Engineering review reference](../../../skills/engineer-review/SKILL.md)
- [Error handling reference](../../../references/utilities/error-handling.md)
- [Hexagonal architecture reference](../../../references/practices/hexagonal-architecture.md)
