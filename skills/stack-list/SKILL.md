---
name: stack-list
description: List active and disabled `references`, `skills`, or `agents` from the canonical stack source of truth (`.specify/stack.yml` when present, `.specify/docs/stack.md` as legacy fallback), with optional type, state, and global-scope filters.
argument-hint: "[type?] [state?] [scope?]"
---

# Stack List

## Objective

List AI artifacts declared in `stack.md`, grouped by active and disabled state, while respecting single-repo and monorepo root precedence.

## When to Use

Use this skill when you need to:
- list all declared artifacts in the current project;
- inspect global defaults in `~/.specify/stack.md`;
- filter by `reference`, `skill`, or `agent`;
- list only `active` or only `disabled` items;
- inspect one package scope in a monorepo while still resolving from the root file.

## Inputs

Optional filters:
- `type` — `reference`, `skill`, or `agent`
- `state` — `active`, `disabled`, or `all` (default: `all`)
- `scope` — `global`, `root`, or a package path such as `packages/app`

Plural aliases (`references`, `skills`, `agents`) may be accepted as input, but they must be normalized internally to the singular form.

## Responsibilities

### 1. Resolve the canonical stack source

**Resolution order (YAML-first):**

1. If `scope=global` → `~/.specify/stack.yml`; fallback to `~/.specify/stack.md` if YAML does not exist
2. If `.specify/stack.yml` exists in the project → use it (YAML is the source of truth)
3. If only `.specify/docs/stack.md` exists → use it (legacy behavior preserved)
4. If neither exists → fallback to `~/.specify/stack.yml` → `~/.specify/stack.md`

**Monorepo detection:**
- Detect the monorepo root from signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces`
- In a monorepo, use `<root>/.specify/stack.yml` (or `<root>/.specify/docs/stack.md` as fallback)
- Report the resolved file as the `source`

### 2. Parse the resolved source

**If the source is `.specify/stack.yml`** — read the YAML structure:
- `references.active` and `references.disabled`
- `skills.active` and `skills.disabled`
- `agents.active` and `agents.disabled`
- For scoped entries (`{ id, scope }`), extract both fields

**If the source is `stack.md`** — read the following Markdown sections:
- `## Active References`
- `## Disabled References`
- `## Active Skills`
- `## Disabled Skills`
- `## Active Agents`
- `## Disabled Agents`

Only explicitly declared items are listed. Undeclared artifacts remain implicit and are not returned by default.

### 3. Apply filters

- `type` narrows results to `reference`, `skill`, or `agent`
- `state` narrows results to `active`, `disabled`, or both
- `scope=global` lists entries from `~/.specify/stack.md`
- `scope` as a package path narrows results to matching scoped entries, while still resolving from the root file in monorepos

### 4. Return grouped results with counts

The result must be easy to scan and include the resolved source file.

## Output Format

```md
Source: `<resolved-stack-md-path>`
Mode: global | single-repo | monorepo
Filters: `type=<value>`, `state=<value>`, `scope=<value>`

## Active References (N)
- `...`

## Disabled References (N)
- `...`
```

When a filter removes all results, return an explicit empty-state message instead of implying success silently.

## Guardrails

- Always resolve from the canonical stack source using the YAML-first precedence rule
- `~/.specify/stack.yml` stores global user defaults; repository-local files override it within a project
- In monorepos, the root stack file overrides package-level files
- Never use the legacy term `knowledge`; use `references`
- If a package-level file conflicts with the root, mention that the derived view is out of sync
- If both `stack.yml` and `stack.md` exist in a project, read only from `stack.yml` and note that `stack.md` is a legacy artifact

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
