---
name: engineer-review
description: Reviews code, diffs, or modules for bugs, risks, architecture alignment, and production concerns. Triggered when the user asks to review code, check a diff, audit a module, evaluate a PR, or validate implementation quality.
argument-hint: "[file path, diff, or module description]"
allowed-tools: Read, Glob, Grep, Bash
---

# Engineering Review

## Objective

Produce a structured, actionable review of the provided code, diff, or module. Detect real issues — not style preferences. Prioritize production risks, correctness, and architectural alignment over cosmetic concerns.

## When to Use

Use this skill when the user asks to:
- Review code before merging or deploying
- Evaluate a diff or PR
- Audit a module for risks or debt
- Check implementation quality against a spec
- Validate that a change is safe to ship

## Inputs

- Code snippet, file path, or diff
- (Optional) Spec or acceptance criteria from `./.specify/specs`
- (Optional) Target context: production deploy, architecture review, security audit

## Responsibilities

### 1. Read and Understand Before Commenting
Read the full context — not just the changed lines. Check callers, types, interfaces, and tests. Never review in isolation.

### 2. Detect Critical Issues
Look for:
- Logic errors that produce wrong results
- Null/undefined access without guard
- Missing error handling or silent swallowing
- Race conditions or concurrency bugs
- Security vulnerabilities (injection, auth bypass, PII leakage)
- Data integrity risks (missing transactions, partial writes)
- Infinite loops, missing termination conditions

### 3. Evaluate Architecture Alignment
Check against `./.specify/specs` if available:
- Does the implementation match the spec?
- Does it follow the dependency flow: domain ← usecases ← adapters?
- Does it introduce coupling that violates clean/hexagonal architecture?
- Are infrastructure concerns leaking into domain logic?

### 4. Assess Maintainability
- Functions over 50 lines should be flagged
- Complex conditionals without extracted named conditions
- Magic strings or numbers without constants
- Test coverage — are critical paths covered?

### 5. Consider Production Concerns
- External calls without timeout
- No retry or circuit breaker on critical paths
- Missing observability (no logs on failures)
- Hardcoded configs or secrets

## Output Format

```
## Engineering Review

### Critical Issues
[Issues that must be fixed before shipping. Each with: location, problem, suggested fix]

### Improvements
[Issues that should be addressed but won't cause immediate failure]

### Strengths
[What was done well — not filler, only genuine observations]

### Trade-offs
[Decisions where the approach has legitimate pros/cons worth discussing]

### Next Steps
[Prioritized list of recommended actions]
```

## Quality Bar

A review is complete when:
- Every critical issue is specific (file:line, not vague)
- Each issue has a suggested fix or direction
- No false positives from incorrect assumptions about the codebase
- The reviewer has actually read the referenced code, not summarized from signatures

## References to Consult

- `references/utilities/error-handling.md` — error handling patterns and anti-patterns
- `references/utilities/testing.md` — test coverage expectations and review criteria
- `references/utilities/logging.md` — observability and log quality
- `references/practices/hexagonal-architecture.md` — architecture rules and layer boundaries
- `references/practices/api-design.md` — API contract quality and REST conventions
- `references/languages/typescript.md` — TypeScript patterns (load if detected in stack)
- `references/languages/go.md` — Go patterns (load if detected in stack)
- `references/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `references/frameworks/react.md` — React patterns (load if detected in stack)
- `references/frameworks/ionic-react.md` — Ionic React patterns (load if detected in stack)
- `references/frameworks/capacitor.md` — Capacitor native bridge patterns (load if detected in stack)
- `references/libs/prisma.md` — Prisma/ORM patterns (load if detected in stack)
- `references/libs/axios.md` — HTTP client patterns (load if detected in stack)
- `references/libs/react-router-dom.md` — React Router v5 patterns (load if detected in stack)
- `references/libs/vite.md` — Vite build tool patterns (load if detected in stack)
- `references/libs/ionicons.md` — Ionicons accessibility and tree-shaking (load if detected in stack)

## Repository Areas to Inspect

- `./.specify/specs` — check if implementation matches spec
- Test files adjacent to reviewed code
- Interfaces/ports the code depends on
- Migration files if database changes are involved

## Guardrails

- Do not suggest refactoring unrelated to the reviewed scope
- Do not report style issues as critical
- Do not assume bugs — verify by reading the full execution path
- Do not praise generic patterns — only comment when genuinely notable

## Example

User: "Review this diff — I'm adding a new payment endpoint"

Actions:
1. Read the diff and full file context
2. Check `./.specify/specs` for payment spec if it exists
3. Verify error handling for external payment call
4. Check that domain errors are not leaking HTTP details
5. Verify idempotency handling if payment can be retried
6. Produce structured output with prioritized findings
