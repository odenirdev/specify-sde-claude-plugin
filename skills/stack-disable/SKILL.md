---
name: stack-disable
description: Disable a `reference`, `skill`, or `agent` by removing it from `active` in the canonical `.specify/stack.yml`. Supports plugin-level `config/index.yml`, single-repo, and monorepo root precedence.
argument-hint: "[type] [artifact] [scope?]"
---

# Stack Disable

## Objective

Disable an AI artifact by removing it from `active` in the canonical `.specify/stack.yml` file. Absence from `active` implies inactive — there is no `disabled` list.

## When to Use

Use this skill when you need to:
- disable a `reference`, `skill`, or `agent`;
- remove an artifact from the active set without deleting its definition;
- manage plugin defaults in `config/index.yml`;
- record a package-scoped disable in a monorepo without letting package docs override the root.

## Inputs

- `type` — `reference`, `skill`, or `agent`
- `artifact` — canonical identifier
- `scope` (optional) — `global`, `root`, or a package path such as `packages/functions`

Plural aliases (`references`, `skills`, `agents`) may be accepted as input, but they must be normalized internally to the singular form.

## Responsibilities

### 1. Resolve the canonical `stack.yml`

- If `scope=global`, use `<plugin-root>/config/index.yml`
  - If file does not exist → return: "Plugin config not found at `config/index.yml`. Run stack-init to configure."
- If no scope, use `.specify/stack.yml` in the project root
  - If file does not exist → return: "Project stack not configured. Run stack-init to initialize `.specify/stack.yml`."
- No fallback between scopes — each resolves independently
- In a monorepo, detect the root from signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces`, then use `<root>/.specify/stack.yml`
- Package-level `stack.yml` files are derived views only

### 2. Normalize and validate the request

- Normalize `type` to one of `reference`, `skill`, or `agent`
- Validate the artifact identifier format
- Normalize the scope to `global` when explicitly requested; otherwise use `root` when operating inside a project

### 3. Update the managed YAML keys

- Remove the artifact from `<type>.active` for the same scope, if present
- If the artifact was not in `active`, treat as a no-op and report accordingly
- Never add a `disabled` key — absence from `active` is the only representation of inactive state
- For scoped entries (monorepo), match and remove the inline object `{ id: "<id>", scope: "<package-path>" }` exactly

### 4. Keep the operation idempotent

If the artifact is already absent from `active` for the same scope, return a no-op style confirmation.

## Output Format

```md
Result: updated | no-op
Action: disable
Type: reference | skill | agent
Artifact: `<canonical-id>`
Scope: global | root | `<package-path>`
Source: `<resolved-stack-yml-path>`
State: inactive (removed from active)
```

## Guardrails

- Use `.specify/stack.yml` as the only source of truth for artifact state
- `config/index.yml` stores plugin defaults; `.specify/stack.yml` in the project is the project-level source of truth — they are independent and do not override each other
- In monorepos, write scoped entries in the **root** `stack.yml`
- Never use the legacy term `knowledge`; use `references`
- Never write a `disabled` key — only `active` exists; absence implies inactive

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
