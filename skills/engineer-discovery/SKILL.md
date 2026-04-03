---
name: engineer-discovery
description: Conducts a structured interview with the user to capture feature requirements and generates prd.md (Product Requirements Document). Triggered when the user wants to start a new feature, capture requirements before defining a spec, run the discovery phase, or create a prd for a feature.
argument-hint: "[feature name or problem description]"
allowed-tools: Read, Glob, Grep, Write
---

# Engineer Discovery

## Objective

Conduct a structured requirements interview with the user and produce a `prd.md` that captures the problem, goals, users, functional and non-functional requirements, acceptance criteria, and out-of-scope items. The output must be complete enough to serve as direct input to `engineer-define-spec`.

## When to Use

Use this skill when the user wants to:
- Start a new feature or initiative from scratch
- Capture requirements before generating a spec
- Create a structured PRD from a rough idea or problem statement
- Run the discovery phase of the engineering pipeline

## Inputs

- Feature name or rough description provided by the user
- (Optional) Existing context in `./.specify/specs/` — read to detect related features and avoid duplication

## Responsibilities

### 1. Scan Existing Context

Before starting the interview:
- Check `./.specify/specs/` for existing features that may overlap
- Read `./.specify/docs/` if present for project conventions and architectural constraints
- Note any existing patterns or constraints to surface during the interview

### 2. Conduct the Interview

Ask the user the following questions in sequence. Wait for each answer before proceeding:

1. **Problem** — What problem are we solving? Who experiences it and when?
2. **Goal** — What does success look like? What measurable outcome do we want?
3. **Users** — Who are the primary users? Any secondary users or systems involved?
4. **Functional Requirements** — What must the system do? List the core behaviors.
5. **Non-Functional Requirements** — Performance, security, scalability, availability constraints?
6. **Out of Scope** — What is explicitly excluded from this iteration?
7. **Acceptance Criteria** — How do we verify the feature is done? What tests or behaviors prove it works?

Do not batch all questions at once unless the user has already provided enough context to answer them directly.

### 3. Derive the Feature Slug

From the feature name, derive a kebab-case slug (e.g., `user-auth`, `payment-flow`, `notification-service`). Confirm with the user if ambiguous.

### 4. Write prd.md

Write `./.specify/specs/[slug]/prd.md` using the output format below. If the directory does not exist, create it. If the file already exists, overwrite it and refresh `updated_at`.

## Output Format

```markdown
---
updated_at: YYYY-MM-DDTHH:MM:SSZ
---

# PRD: [Feature Name]

## Problem

[Clear description of the problem being solved, who experiences it, and when]

## Goals

- [Measurable goal 1]
- [Measurable goal 2]

## Users

- **Primary**: [Who directly uses this feature]
- **Secondary**: [Other users or systems involved, if any]

## Functional Requirements

- [FR-1] [What the system must do]
- [FR-2] [What the system must do]

## Non-Functional Requirements

- [NFR-1] [Performance, security, scalability, or availability constraint]

## Out of Scope

- [What is explicitly not included in this iteration]

## Acceptance Criteria

- [ ] [Verifiable condition 1]
- [ ] [Verifiable condition 2]
```

## Output Location

Write to: `./.specify/specs/[feature-slug]/prd.md`

The file must include the `updated_at` frontmatter. If the file already exists, overwrite and refresh `updated_at`.

## Quality Bar

A PRD is complete when:
- The problem is stated in terms of user impact, not solution
- Goals are measurable or verifiable
- All functional requirements are explicit — no ambiguous "should support" language
- Acceptance criteria can be translated directly into tests
- Out of scope is explicitly listed, not assumed

## Knowledge to Consult

- `knowledge/practices/documentation-derivation.md` — writing accurate, evidence-based content
- `knowledge/practices/task-breakdown.md` — understanding what makes requirements actionable

## Guardrails

- Do not skip the interview — do not invent requirements without asking
- Do not include implementation details in the PRD (no tech stack, no architecture decisions)
- Do not leave acceptance criteria empty or vague
- Do not conflate goals with requirements — goals describe outcomes, requirements describe behavior

## Example

User: "I want to add user notifications to our app"

Actions:
1. Check `./.specify/specs/` — no existing notification feature found
2. Ask: "What problem are we solving?" → "Users miss important events because there's no alert system"
3. Ask: "What's the goal?" → "Reduce missed events by 80% within 30 days of launch"
4. Ask: "Who are the users?" → "All authenticated users; admin can configure notification types"
5. Ask: "What must the system do?" → "Send in-app, email, and push notifications for specific event types"
6. Ask: "Any NFRs?" → "Notifications must be delivered within 5 seconds of the event"
7. Ask: "Out of scope?" → "SMS notifications, notification scheduling"
8. Ask: "Acceptance criteria?" → "User receives in-app notification within 5s of a new message; unread count badge updates"
9. Derive slug: `user-notifications`
10. Write `./.specify/specs/user-notifications/prd.md`
