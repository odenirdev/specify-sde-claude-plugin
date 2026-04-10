---
name: review-triage
description: Analyze a single review topic (PR comment, code review feedback, or issue) by identifying root cause, impact, and proposing a safe fix. Waits for user confirmation before implementing. Use when the user shares a review item and wants a structured analysis before acting.
argument-hint: "[review comment, PR feedback, or issue description]"
allowed-tools: Read, Glob, Grep, Bash
---

# Review Triage

## Objective

Analyze a single review topic — a PR comment, code review finding, or reported issue — and produce a structured triage: root cause, impact, and a safe proposed fix. Do NOT implement anything until the user confirms.

## When to Use

- User pastes a code review comment and wants to understand it before acting
- User shares a PR finding and wants a safe fix proposal
- User wants to triage a bug or issue with cause + impact analysis
- User runs `/review-triage` with a specific topic

## Inputs

- Review comment, PR feedback snippet, or issue description (required)
- File path or code context (optional — grep/glob if not provided)

## Procedure

### Step 1 — Understand the topic

Read the review comment carefully. If it references a file or line, read that file with full context (not just the flagged line). Check callers, types, and tests if relevant.

### Step 2 — Root cause analysis

Identify the underlying cause of the issue. Ask:
- Is this a logic error, missing guard, incorrect assumption, or design problem?
- Is the root cause in the flagged location or upstream?
- Is this a symptom of a broader pattern?

### Step 3 — Impact assessment

Determine the blast radius:
- **Severity**: Critical / High / Medium / Low
- **Scope**: isolated change vs. cascading risk
- **User-facing**: does this affect end users or is it internal?
- **Data risk**: can this cause data loss, corruption, or security exposure?

### Step 4 — Propose a safe fix

Provide ONE concrete, minimal fix that:
- Addresses the root cause, not just the symptom
- Doesn't introduce new risk or scope creep
- Can be reviewed and reverted independently
- Follows existing code conventions in the file

If multiple approaches exist, list them with a clear recommendation and trade-offs.

### Step 5 — Wait for confirmation

Present the analysis and ask:

> **Ready to implement?** Reply `yes` to apply the fix, `no` to discard, or describe any adjustments before proceeding.

Do NOT implement until the user explicitly confirms. If the user wants to post this as a PR comment instead, offer the formatted text.

## Output Format

```
## Review Triage

**Topic:** [one-line summary of the review item]

### Root Cause
[Specific explanation of why this exists. File:line if applicable.]

### Impact
- **Severity**: Critical / High / Medium / Low
- **Scope**: [what is affected]
- **Risk**: [data, security, UX, or correctness concern]

### Proposed Fix
[Concrete, minimal change. Include code snippet if < 20 lines.]

**Why this fix is safe:**
[Brief rationale — why it won't break anything else]

**Alternative approaches (if applicable):**
- Option A: [trade-off]
- Option B: [trade-off]

---
Ready to implement? Reply `yes` to apply, `no` to skip, or describe adjustments.
```

## Guardrails

- Analyze ONE topic per run. If the user provides multiple, pick the first and ask to re-run for others.
- Never implement before explicit confirmation — not even a "small" fix.
- Do not refactor unrelated code while fixing the flagged item.
- If the root cause is unclear after reading the code, say so explicitly and ask a targeted question.
- Do not invent impacts that aren't grounded in the actual code path.

## PR Comment Mode

If the user asks to post this as a PR comment instead of implementing, produce the output in this format:

```
**[Review Triage]** — [Topic summary]

**Root Cause:** [one sentence]
**Impact:** [severity + scope]
**Proposed Fix:** [brief description or inline code block]

Waiting for confirmation before implementing.
```

## References

- [Error handling reference](../../references/utilities/error-handling.md)
- [Testing reference](../../references/utilities/testing.md)
- [Hexagonal architecture reference](../../references/practices/hexagonal-architecture.md)

## Example

User: `/review-triage` — "Missing null check on `user.profile` before accessing `user.profile.avatar`"

Actions:
1. Grep for `user.profile.avatar` usage in the codebase
2. Read the full function and callers to understand when `profile` can be null
3. Identify root cause: `getProfile()` can return null on new accounts
4. Assess impact: Medium — crashes the component for new users, not a security issue
5. Propose fix: `user.profile?.avatar` optional chaining, or guard at the fetch boundary
6. Present structured analysis and wait for `yes` before touching any file
