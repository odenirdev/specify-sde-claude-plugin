---
name: stack-enable
description: Enable a `reference`, `skill`, or `agent` by updating the canonical `.specify/stack.yml` source of truth. Supports plugin-level `config/index.yml`, single-repo, and monorepo root precedence.
argument-hint: "[type] [artifact] [scope?]"
---

# Stack Enable

## Objective

Enable an AI artifact by marking it as active in the canonical `.specify/stack.yml` file.

## When to Use

Use this skill when you need to:
- enable a `reference`, `skill`, or `agent`;
- add an artifact that is not yet in `active`;
- manage plugin defaults in `config/index.yml`;
- record a package-scoped activation in a monorepo while keeping the root file as the source of truth.

## Inputs

- `type` — `reference`, `skill`, or `agent`
- `artifact` — canonical identifier
- `scope` (optional) — `global`, `root`, or a package path such as `packages/app`

Plural aliases (`references`, `skills`, `agents`) may be accepted as input, but they must be normalized internally to the singular form.

## Responsibilities

### 1. Resolve the canonical `stack.yml`

- If `scope=global`, use `<plugin-root>/config/index.yml`
  - If file does not exist → return: "Plugin config not found at `config/index.yml`. Run stack-init to configure."
- If no scope, use `.specify/stack.yml` in the project root
  - If file does not exist → return: "Project stack not configured. Run stack-init to initialize `.specify/stack.yml`."
- No fallback between scopes — each resolves independently
- In a monorepo, detect the root from signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces`, then use `<root>/.specify/stack.yml`
- Do **not** treat package-level `stack.yml` files as the source of truth

### 2. Normalize and validate the request

- Normalize `type` to one of `reference`, `skill`, or `agent`
- Validate the artifact identifier format:
  - `reference` → repo-relative path under `references/`
  - `skill` → skill name/folder slug
  - `agent` → agent slug/name
- Normalize the scope to `global` when explicitly requested; otherwise use `root` when operating inside a project

### 3. Update the managed YAML keys

- Ensure the matching `<type>.active` key exists in the YAML
- Add the artifact to `<type>.active` if it is not already there
- Never duplicate an entry
- If the artifact was undeclared, add it explicitly as active
- For scoped entries (monorepo), use the inline object format: `{ id: "<id>", scope: "<package-path>" }`

### 4. Keep the operation idempotent

If the artifact is already active for the same scope, return a no-op style confirmation instead of rewriting duplicate entries.

## Output Format

```md
Result: updated | no-op
Action: enable
Type: reference | skill | agent
Artifact: `<canonical-id>`
Scope: global | root | `<package-path>`
Source: `<resolved-stack-yml-path>`
State: active
```

## Guardrails

- Use `.specify/stack.yml` as the only source of truth for artifact state
- `config/index.yml` stores plugin defaults; `.specify/stack.yml` in the project is the project-level source of truth — they are independent and do not override each other
- In monorepos, write scoped entries in the **root** `stack.yml`
- Never use the legacy term `knowledge`; use `references`
- Only `active` exists — absence from `active` implies inactive

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
