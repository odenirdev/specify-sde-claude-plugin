---
name: engineer-debug
description: Performs root cause analysis on bugs, failures, or unexpected behavior. Produces a structured debugging strategy with hypotheses, investigation steps, and evidence to collect. Triggered when the user reports a bug, unexpected behavior, test failure, or production incident.
argument-hint: "[error message, symptom description, or failing test]"
allowed-tools: Read, Glob, Grep, Bash
---

# Engineering Debug

## Objective

Identify the most likely root cause of a bug or failure and produce a concrete debugging strategy. Do not guess — form hypotheses based on evidence, then validate them systematically.

## When to Use

Use this skill when the user reports:
- A bug or unexpected behavior
- A failing test
- A production error or incident
- An integration failure
- Performance degradation

## Inputs

- Error message, stack trace, or symptom description
- (Optional) Failing test output
- (Optional) Logs, metrics, or timeline of events
- (Optional) Recent changes that may be related

## Responsibilities

### 1. Understand the Symptom Precisely
Before forming hypotheses, clarify:
- What is the exact behavior observed?
- What is the expected behavior?
- When did it start? After what change?
- Is it reproducible? Under what conditions?

### 2. Read the Relevant Code
Read the full execution path, not just the line that throws:
- Entry point (HTTP handler, queue consumer, cron)
- Through use cases / business logic
- Down to infrastructure (DB queries, external calls)
- Check error handling at each boundary

### 3. Form Hypotheses
Generate 3–5 candidate root causes ordered by probability:
- Most likely: the simplest explanation supported by the evidence
- Consider: timing issues, data state, config/env, dependency version, edge case input

### 4. Define Investigation Steps
For each hypothesis, define what evidence would confirm or rule it out:
- What to log and where
- What to inspect (database state, queue messages, external service response)
- What to test in isolation
- What to diff (recent changes, config, dependencies)

### 5. Identify Quick Wins
Are there fast checks that rule out common causes before deeper investigation?
- Is the service running? Is the database reachable?
- Is the environment correct?
- Is this a known issue in the dependency?

## Output Format

```
## Debug Analysis: [Problem Summary]

### Symptom
[Precise description of what is happening vs what should happen]

### Context
[Relevant code paths, dependencies, recent changes]

### Hypotheses
1. **[Most likely cause]** — [Why this is likely based on evidence]
2. **[Second candidate]** — [Why and what would distinguish it from #1]
3. **[Third candidate]** — [Less likely but worth ruling out]

### Debugging Steps
**Step 1 — [Quick check]**
- What to do: [Specific action]
- Expected result if hypothesis is correct: [...]
- Expected result if not: [...]

**Step 2 — [Investigation]**
...

### Evidence to Collect
- [ ] [Specific log entry or metric to look for]
- [ ] [Database query to run]
- [ ] [Endpoint to test]

### Instrumentation Needed
[Any logging, tracing, or debug flags to add to gather more data]
```

## Quality Bar

A debug analysis is complete when:
- The symptom is precisely described (not paraphrased)
- Hypotheses are ordered by probability with reasoning
- Steps are specific enough to execute without clarification
- Quick, low-cost checks are listed first

## References to Consult

- `references/utilities/error-handling.md` — common error handling bugs and propagation
- `references/utilities/logging.md` — what to log for traceability and correlation
- `references/utilities/testing.md` — test isolation to understand expected behavior
- `references/practices/hexagonal-architecture.md` — layer boundaries to trace the execution path
- `references/practices/api-design.md` — expected API contract to identify deviations
- `references/languages/typescript.md` — TypeScript patterns (load if detected in stack)
- `references/languages/go.md` — Go patterns (load if detected in stack)
- `references/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `references/frameworks/react.md` — React patterns (load if detected in stack)
- `references/frameworks/ionic-react.md` — Ionic lifecycle and routing bugs (load if detected in stack)
- `references/frameworks/capacitor.md` — Capacitor native bridge and plugin failures (load if detected in stack)
- `references/libs/prisma.md` — Prisma/ORM patterns (load if detected in stack)
- `references/libs/axios.md` — HTTP client patterns (load if detected in stack)
- `references/libs/react-router-dom.md` — React Router v5 navigation bugs (load if detected in stack)
- `references/libs/vite.md` — Vite build and HMR issues (load if detected in stack)
- `references/libs/ionicons.md` — Ionicons rendering and tree-shaking bugs (load if detected in stack)

## Repository Areas to Inspect

- The full execution path from entry to failure point
- Recent git changes in the affected module
- Related tests to understand expected behavior
- Config/env files for environment-specific issues

## Guardrails

- Do not assume the first obvious cause is correct — always list alternatives
- Do not suggest adding logs that expose sensitive data
- Do not suggest fixes before identifying the root cause
- Do not say "this looks like a race condition" without evidence — describe what would prove it

## Example

User: "The order webhook is randomly failing with a 500 — no error in logs"

Actions:
1. Read the webhook handler and trace the full execution path
2. Check error handling — is something catching and swallowing the error before logging?
3. Form hypotheses: missing error log, async exception not caught, external call timeout
4. Step 1: Add error logging to the catch block if missing
5. Step 2: Check if the external call has a timeout configured
6. Step 3: Reproduce with a specific payload to isolate
7. Produce structured debug analysis
