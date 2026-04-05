---
name: stack-disable
description: Disable a `reference`, `skill`, or `agent` by updating the canonical `stack.md` source of truth. Supports global `~/.specify/stack.md`, single-repo, and monorepo root precedence.
argument-hint: "[type] [artifact] [scope?]"
---

# Stack Disable

## Objective

Disable an AI artifact by marking it as disabled in the canonical `stack.md` file.

## When to Use

Use this skill when you need to:
- disable a `reference`, `skill`, or `agent`;
- keep an artifact explicitly unavailable while preserving it in the declared stack;
- manage global defaults in `~/.specify/stack.md`;
- record a package-scoped disable in a monorepo without letting package docs override the root.

## Inputs

- `type` — `reference`, `skill`, or `agent`
- `artifact` — canonical identifier
- `scope` (optional) — `global`, `root`, or a package path such as `packages/functions`

Plural aliases (`references`, `skills`, `agents`) may be accepted as input, but they must be normalized internally to the singular form.

## Responsibilities

### 1. Resolve the canonical `stack.md`

- If `scope=global`, use `~/.specify/stack.md`
- If no repository-local `stack.md` exists, fall back to `~/.specify/stack.md`
- In a single repo, use `./.specify/docs/stack.md`
- In a monorepo, detect the root from signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces`, then use `<root>/.specify/docs/stack.md`
- Package-level `stack.md` files are derived views only

### 2. Normalize and validate the request

- Normalize `type` to one of `reference`, `skill`, or `agent`
- Validate the artifact identifier format
- Normalize the scope to `global` when explicitly requested; otherwise use `root` when operating inside a project

### 3. Update the managed sections

- Ensure the matching `## Active ...` and `## Disabled ...` sections exist
- Remove the artifact from the matching active section for the same scope, if present
- Add the artifact to the matching disabled section if it is not already there
- Never duplicate an entry
- If the artifact was undeclared, add it explicitly as disabled

### 4. Keep the operation idempotent

If the artifact is already disabled for the same scope, return a no-op style confirmation instead of rewriting duplicate entries.

## Output Format

```md
Result: updated | no-op
Action: disable
Type: reference | skill | agent
Artifact: `<canonical-id>`
Scope: global | root | `<package-path>`
Source: `<resolved-stack-md-path>`
State: disabled
```

## Guardrails

- Use `stack.md` as the only source of truth for artifact state
- `~/.specify/stack.md` stores global user defaults; repository-local files override it within a project
- In monorepos, write scoped entries in the **root** `stack.md`
- Never use the legacy term `knowledge`; use `references`
- Do not leave an artifact in both `Active` and `Disabled` for the same scope

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
