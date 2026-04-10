---
name: stack-list
description: List active `references`, `skills`, or `agents` from the canonical `.specify/stack.yml` source of truth, with optional type and scope filters.
argument-hint: "[type?] [state?] [scope?]"
---

# Stack List

## Objective

List AI artifacts declared as active in `.specify/stack.yml`, while respecting single-repo and monorepo root precedence. Artifacts absent from `active` are implicitly inactive and not shown by default.

## When to Use

Use this skill when you need to:
- list all active artifacts in the current project;
- inspect plugin defaults in `config/index.yml`;
- filter by `reference`, `skill`, or `agent`;
- inspect one package scope in a monorepo while still resolving from the root file.

## Inputs

Optional filters:
- `type` — `reference`, `skill`, or `agent`
- `scope` — `global`, `root`, or a package path such as `packages/app`

Plural aliases (`references`, `skills`, `agents`) may be accepted as input, but they must be normalized internally to the singular form.

## Responsibilities

### 1. Resolve the canonical `stack.yml`

**Resolution order:**

1. If `scope=global` → `<plugin-root>/config/index.yml`
   - If file does not exist → return: "Plugin config not found at `config/index.yml`. Run stack-init to configure."
2. If no scope → `.specify/stack.yml` in the project root
   - If file does not exist → return: "Project stack not configured. Run stack-init to initialize `.specify/stack.yml`."

No fallback between scopes. Each scope resolves independently.

**Monorepo detection:**
- Detect the monorepo root from signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces`
- In a monorepo, use `<root>/.specify/stack.yml`
- Report the resolved file as the `source`

### 2. Parse the resolved `stack.yml`

Read the YAML structure:
- `references.active`
- `skills.active`
- `agents.active`
- For scoped entries (`{ id, scope }`), extract both fields

Only items explicitly declared in `active` are listed. Absence implies inactive.

### 3. Apply filters

- `type` narrows results to `reference`, `skill`, or `agent`
- `scope=global` lists entries from `<plugin-root>/config/index.yml`
- `scope` as a package path narrows results to matching scoped entries, while still resolving from the root file in monorepos

### 4. Return grouped results with counts

The result must be easy to scan and include the resolved source file.

## Output Format

```md
Source: `<resolved-stack-yml-path>`
Mode: global | single-repo | monorepo
Filters: `type=<value>`, `scope=<value>`

## References (N)
- `...`

## Skills (N)
- `...`

## Agents (N)
- `...`
```

When a filter removes all results, return an explicit empty-state message instead of implying success silently.

## Guardrails

- Always resolve from `.specify/stack.yml` as the single source of truth
- `config/index.yml` stores plugin defaults; `.specify/stack.yml` in the project is the project-level source of truth — they are independent and do not override each other
- In monorepos, the root stack file overrides package-level files
- Never use the legacy term `knowledge`; use `references`
- Only `active` exists — do not infer or display a `disabled` list
- If a package-level file conflicts with the root, mention that the derived view is out of sync

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
