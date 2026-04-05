---
name: stack-list
description: List active and disabled `references`, `skills`, or `agents` from the canonical `stack.md` source of truth, with optional type, state, and global-scope filters.
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

### 1. Resolve the canonical `stack.md`

- If `scope=global`, use `~/.specify/stack.md`
- If no repository-local `stack.md` exists, fall back to `~/.specify/stack.md`
- In a single repo, use `./.specify/docs/stack.md`
- In a monorepo, detect the root from signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces`, then use `<root>/.specify/docs/stack.md`
- Report the resolved file as the `source`

### 2. Parse the managed sections

Read the following sections when present:
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

- Always resolve from the canonical `stack.md`
- `~/.specify/stack.md` stores global user defaults; repository-local files override it within a project
- In monorepos, root `stack.md` overrides package-level files
- Never use the legacy term `knowledge`; use `references`
- If a package-level file conflicts with the root, mention that the derived view is out of sync

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
