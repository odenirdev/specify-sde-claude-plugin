---
name: stack-init
description: Initialize `.specify/stack.yml` for a project. Migrates from `stack.md` when present, or creates a minimal YAML from scratch. Establishes the declarative stack source of truth consumed by stack-enable, stack-disable, and stack-list.
argument-hint: "[--migrate] [--force]"
---

# Stack Init

## Objective

Create `.specify/stack.yml` as the declarative stack configuration for the current project. This file becomes the primary source of truth for all AI artifact management — which references, skills, and agents are active or disabled.

## When to Use

Use this skill when you need to:
- bootstrap a new project with a `stack.yml` configuration;
- migrate an existing `stack.md` into the YAML format;
- reset the stack configuration after a major restructuring;
- create the file that `stack-enable`, `stack-disable`, and `stack-list` will manage.

## Inputs

Optional flags:
- `--migrate` — force migration mode even if `.specify/stack.yml` already exists
- `--force` — overwrite `.specify/stack.yml` without asking for confirmation

## Responsibilities

### 1. Check for existing configuration

- If `.specify/stack.yml` already exists and `--force` was not provided:
  - Warn the user that the file exists
  - Ask for explicit confirmation before overwriting
  - If the user declines, abort and report no-op

### 2. Detect migration source

Check whether `.specify/docs/stack.md` exists:
- If yes → offer migration mode
- If no → proceed with fresh initialization

### 3. Migration mode (when `stack.md` exists)

Parse the following managed sections from `.specify/docs/stack.md`:
- `## Active References` → `references.active`
- `## Disabled References` → `references.disabled`
- `## Active Skills` → `skills.active`
- `## Disabled Skills` → `skills.disabled`
- `## Active Agents` → `agents.active`
- `## Disabled Agents` → `agents.disabled`

For each entry:
- Strip backticks and inline scope annotations from the MD format
- If the entry has `— scope: <path>`, convert to object format: `{ id: "<id>", scope: "<path>" }`
- Otherwise, keep as a plain string

Write the parsed data into `.specify/stack.yml` using the canonical format (see below).

### 4. Fresh initialization (when no `stack.md` exists)

Create `.specify/stack.yml` with the minimal structure:

```yaml
version: "1"

references:
  active: []
  disabled: []

skills:
  active: []
  disabled: []

agents:
  active: []
  disabled: []
```

### 5. Confirm and report

After writing the file, output a confirmation using the output format below. Do not automatically update or delete `stack.md` — it remains as a legacy view until the user decides to remove it.

## Canonical YAML Format

```yaml
version: "1"

references:
  active:
    - references/practices/hexagonal-architecture.md
    - { id: references/frameworks/react.md, scope: packages/app }
  disabled:
    - references/frameworks/langgraph.md

skills:
  active:
    - stack-list
    - stack-enable
    - stack-disable
    - stack-init
  disabled: []

agents:
  active:
    - reviewer
    - backend-architect
  disabled:
    - langgraph-architect
```

**Format rules:**
- Simple entries: plain string with the canonical identifier
- Scoped entries (monorepo): inline object `{ id: "...", scope: "packages/app" }`
- Empty sections must use `[]`, not be omitted
- `version: "1"` is required at the top level

## Output Format

```md
Result: created | migrated | no-op
Source: `.specify/docs/stack.md` | fresh
Target: `.specify/stack.yml`

## References
Active: N
Disabled: N

## Skills
Active: N
Disabled: N

## Agents
Active: N
Disabled: N
```

## Guardrails

- Never delete or modify `stack.md` — leave it as a legacy artifact for the user to remove
- Never write outside `.specify/stack.yml` without explicit user confirmation
- Preserve all existing entries when migrating — do not infer or invent artifacts
- If a `stack.md` entry is ambiguous or malformed, skip it and report it as a warning
- Follow the canonical YAML format exactly; do not add extra keys or sections

## Spec Reference

Follow the contract in `./.specify/specs/stack-artifact-management/spec.md`.
