---
name: engineer-tradeoff
description: Compares implementation or architectural options with structured pros/cons, operational impact, and a recommendation. Triggered when the user needs to decide between approaches, evaluate alternatives, or justify a technical choice.
argument-hint: "[decision or options to compare]"
allowed-tools: Read, Glob, Grep
---

# Engineering Trade-off

## Objective

Produce a structured comparison of two or more options for a technical decision. The output should be concrete enough to make the decision — not a list of "it depends" hedges. End with a recommendation with clear rationale.

## When to Use

Use this skill when the user wants to:
- Choose between implementation approaches
- Evaluate architectural alternatives
- Justify a technical decision to stakeholders
- Compare libraries, patterns, or strategies
- Understand the long-term implications of a choice

## Inputs

- Description of the decision to make
- Options to compare (or ask to identify them)
- (Optional) Constraints: performance, team experience, timeline, existing system
- (Optional) Relevant spec from `./.specify/specs`

## Responsibilities

### 1. Clarify the Decision
Before comparing, be precise about:
- What is being decided?
- What are the non-negotiable constraints?
- What is the optimization target? (simplicity, performance, scalability, speed to ship)

### 2. Identify All Viable Options
List options with a one-line description. If options weren't provided, identify them from the context.

### 3. Evaluate Each Option
For each option, analyze:
- **Correctness**: Does it actually solve the problem?
- **Complexity**: How hard is it to implement and maintain?
- **Performance**: What are the latency, throughput, and resource implications?
- **Operational cost**: What does it cost to run, monitor, and debug?
- **Risk**: What can go wrong? How does it fail?
- **Reversibility**: How hard is it to change later?

### 4. Apply the Constraints
Filter or rank options against stated constraints. An option that is theoretically better but violates a hard constraint is not viable.

### 5. Recommend
Make a concrete recommendation. "Option A" not "either could work." Include:
- Why this option
- What conditions would make Option B better
- Any mitigation for its weaknesses

## Output Format

```
## Trade-off Analysis: [Decision]

### Context
[What problem this decision addresses and key constraints]

### Options

#### Option A: [Name]
**Summary**: [One sentence]
**Pros**:
- [Specific advantage]
- [Specific advantage]
**Cons**:
- [Specific disadvantage]
- [Specific disadvantage]
**Operational impact**: [How this affects running the system in production]
**Risk**: [What can go wrong]
**Reversibility**: [Easy / Hard / Irreversible — with explanation]

#### Option B: [Name]
...

### Comparison Matrix

| Criterion | Option A | Option B |
|---|---|---|
| Implementation complexity | Low | High |
| Performance | Medium | High |
| Operational cost | Low | Medium |
| Reversibility | Easy | Hard |

### Recommendation

**Choose: Option A**

Because: [2–3 sentences explaining why this is the better choice given the constraints]

**Reconsider if**: [Conditions under which Option B would be the right call]

**Mitigation for main weakness**: [How to address the biggest downside of the recommended option]
```

## Output Location

Write the output to `./.specify/specs/[feature-slug]/tradeoff.md`. If the directory does not exist, create it.

The file must start with:
```
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---
```

If the file already exists, update its content and refresh `updated_at`. Do not append — replace in place.

## Quality Bar

A trade-off analysis is complete when:
- All options are evaluated on the same criteria
- The recommendation is specific and actionable
- Conditions that would change the recommendation are stated
- No option is dismissed without explanation

## Knowledge to Consult

- `knowledge/practices/hexagonal-architecture.md` — architectural decisions and boundaries
- `knowledge/practices/api-design.md` — API design decisions and trade-offs
- `knowledge/utilities/error-handling.md` — error handling trade-offs between approaches
- `knowledge/utilities/testing.md` — testability implications of each option
- `knowledge/languages/typescript.md` — TypeScript patterns (load if detected in stack)
- `knowledge/languages/go.md` — Go patterns (load if detected in stack)
- `knowledge/frameworks/nestjs.md` — NestJS-specific patterns (load if detected in stack)
- `knowledge/frameworks/react.md` — React patterns (load if detected in stack)
- `knowledge/frameworks/ionic-react.md` — Ionic React vs alternatives trade-offs (load if detected in stack)
- `knowledge/frameworks/capacitor.md` — Capacitor vs native trade-offs (load if detected in stack)
- `knowledge/libs/prisma.md` — Prisma/ORM patterns (load if detected in stack)
- `knowledge/libs/axios.md` — HTTP client patterns (load if detected in stack)
- `knowledge/libs/react-router-dom.md` — React Router v5 vs v6 trade-offs (load if detected in stack)
- `knowledge/libs/vite.md` — Vite vs webpack trade-offs (load if detected in stack)
- `knowledge/libs/ionicons.md` — Ionicons usage trade-offs (load if detected in stack)

## Repository Areas to Inspect

- `./.specify/specs` — constraints and requirements that affect the decision
- `./.specify/docs` — existing architectural decisions (ADRs) to maintain consistency
- Existing patterns in the codebase for the technology being evaluated

## Guardrails

- Do not compare more than 3 options without a clear reason — more options dilute the analysis
- Do not recommend the option the user seems to want — recommend the best option for the problem
- Do not list cons without magnitude — a small con and a critical con are not equivalent
- Do not avoid a recommendation with "it depends" — provide the conditional recommendation explicitly

## Example

User: "Should we use REST or GraphQL for the new partner API?"

Actions:
1. Read `./.specify/specs` for partner API requirements
2. Identify constraints: team experience, client types, data complexity
3. Evaluate REST: simple, cacheable, well-understood, less flexible
4. Evaluate GraphQL: flexible queries, more complex infra, N+1 risk
5. Apply constraints: if clients are mobile and need flexible queries, GraphQL; if internal API, REST
6. Produce structured comparison with concrete recommendation
