---
name: review-topic-triage
description: Triages a single PR review topic by analyzing its cause, impact, and producing a safe fix proposal. Waits for user confirmation or PR comment before applying any change. Triggered when the user asks to triage, analyze, or address a specific review comment or finding.
argument-hint: "[review comment, finding description, or file:line reference]"
allowed-tools: Read, Glob, Grep, Bash
---

# Review Topic Triage

## Objective

For a single review topic (comment, finding, or flagged issue), produce a structured triage: root cause, blast radius, and a safe fix proposal. Do NOT apply any change until the user confirms or acknowledges via comment.

## When to Use

Use this skill when the user wants to:
- Understand the root cause of a specific review comment
- Assess the impact of a flagged issue before touching code
- Get a safe fix proposal for a single review finding
- Decide whether a review comment is worth addressing

## Inputs

- Review comment text, or file:line reference
- (Optional) PR diff or surrounding code context
- (Optional) Related spec or acceptance criteria

## Responsibilities

### 1. Read Before Triaging

Read the full file and its callers — not just the flagged line. Understand the surrounding context before forming any opinion.

### 2. Identify Root Cause

Determine the precise source of the issue:
- Is it a logic error, a missing guard, a wrong abstraction, or a style concern?
- Is it introduced by this change or pre-existing?
- Is the reviewer's framing correct, or is there a misunderstanding?

### 3. Assess Impact

Evaluate the blast radius:
- Does it affect correctness, performance, security, or maintainability?
- Is it confined to this file, or does it propagate through callers?
- What is the risk of leaving it as-is vs. changing it?

### 4. Propose a Safe Fix

Draft a minimal, targeted fix:
- Change only what is necessary to address the finding
- Do not refactor unrelated code
- Flag any side effects or trade-offs the fix introduces
- If multiple approaches exist, present the safest default and name the alternatives

### 5. Wait for Confirmation

**Do NOT apply any code change.** Present the triage output and explicitly ask:
> "Confirma esta abordagem ou tem algum comentário antes de aplicar?"

Apply the fix only after the user explicitly confirms (or comments approval in the PR thread).

## Output Format

```
## Triage: [Short topic title]

### Review Comment
> [Quoted or paraphrased original finding]

### Root Cause
[Precise explanation of why the issue exists — file:line if applicable]

### Impact
- **Severity**: Critical / High / Medium / Low / Style
- **Scope**: [Confined / Propagates to callers / Cross-cutting]
- **Risk if left as-is**: [What breaks or degrades]

### Fix Proposal
[Minimal, targeted description of the change — no code yet]

**Trade-offs / Side effects**:
- [Any risk, regression surface, or downstream impact]

**Alternatives considered**:
- [Option B — why it was not chosen as default]

---
Confirma esta abordagem ou tem algum comentário antes de aplicar?
```

## Quality Bar

A triage is complete when:
- The root cause is specific (file:line when applicable), not generic
- Impact severity is justified by evidence, not estimated
- The fix proposal is minimal — it solves the finding without scope creep
- No code has been written or modified before confirmation

## Guardrails

- Do not apply fixes speculatively — always gate on confirmation
- Do not expand scope: one triage = one finding
- Do not dismiss reviewer comments without reading the full context
- Do not mark a finding as "style only" without verifying there is no semantic effect
- If the reviewer's comment appears to be based on a misunderstanding, say so explicitly and politely

## References

- [Error handling reference](../../references/utilities/error-handling.md)
- [Testing reference](../../references/utilities/testing.md)
- [Hexagonal architecture reference](../../references/practices/hexagonal-architecture.md)

## Example

User: "Triage this review comment: `getUser` may return undefined but it's used without null check on line 42"

Actions:
1. Read `getUser` signature and implementation
2. Read line 42 and its callers
3. Determine if undefined is possible in practice
4. Assess impact: runtime crash vs. type gap only
5. Propose minimal fix: add null guard at line 42 or update return type
6. Present triage and wait for confirmation before touching any file
