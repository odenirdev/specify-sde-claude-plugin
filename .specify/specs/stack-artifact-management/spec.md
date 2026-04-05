---
updated_at: 2026-04-05T00:00:00Z
---

# Spec: Stack-Based AI Artifact Management

## Problem

Projects using `specify-sde` need a standard way to enable, disable, and list AI artifacts (`references`, `skills`, and `agents`). Today, `stack.md` is already the natural place to describe the detected stack, but there is no explicit contract for:

- enabling or disabling artifacts by type;
- listing active vs disabled artifacts;
- filtering list results by type;
- handling undeclared artifacts consistently; and
- resolving precedence in monorepos where both root and package docs exist.

This creates ambiguity across single-repo and monorepo projects and makes artifact management harder to automate safely.

## Goal

Define a clear, implementation-ready contract for three skills:

- `stack-enable`
- `stack-disable`
- `stack-list`

These skills must treat `stack.md` as the source of truth for artifact state, support a global user-level stack at `~/.specify/stack.md`, and follow a root-first precedence model in monorepos.

## Scope

**In scope**
- Standardize the three skills and what each one does
- Define accepted inputs and supported actions
- Define the canonical read/write format in `stack.md`
- Define global, single-repo, and monorepo resolution rules
- Define default behavior for undeclared artifacts
- Define the expected output for list operations
- Provide examples for global, single-repo, and monorepo usage

**Out of scope**
- Automatic artifact discovery beyond what is explicitly declared in `stack.md`
- Backward compatibility with the legacy term `knowledge`
- Runtime-specific installation logic for Copilot or Claude adapters

## Canonical Skill Set

| Skill | Purpose | Supported artifact types |
|---|---|---|
| `stack-enable` | Mark an artifact as active in `stack.md` | `reference`, `skill`, `agent` |
| `stack-disable` | Mark an artifact as disabled in `stack.md` | `reference`, `skill`, `agent` |
| `stack-list` | Read `stack.md` and list declared artifacts by state | `reference`, `skill`, `agent` |

## Supported Inputs

### Common input fields

All three skills support the following normalized fields:

- `type`: one of `reference`, `skill`, `agent`
  - the implementation may accept plural aliases (`references`, `skills`, `agents`) but must normalize to the singular form internally;
- `artifact`: the canonical artifact identifier;
- `scope` (optional): one of:
  - `global` → target `~/.specify/stack.md`;
  - `root` → target the current repository root `stack.md`;
  - `<package-path>` → target one package or sub-project logically, while still resolving writes in the root `stack.md` for monorepos.

### Canonical artifact identifiers

- **Reference** → repository-relative path, e.g. `references/practices/hexagonal-architecture.md`
- **Skill** → skill folder name, e.g. `docs-sync`, `stack-enable`
- **Agent** → agent name/slug, e.g. `reviewer`, `langgraph-architect`

### Action-specific inputs

#### `stack-enable`

Required:
- `type`
- `artifact`

Optional:
- `scope`

#### `stack-disable`

Required:
- `type`
- `artifact`

Optional:
- `scope`

#### `stack-list`

Optional:
- `type`
- `state`: `active`, `disabled`, or `all` (default: `all`)
- `scope`

## `stack.md` as the Source of Truth

The canonical configuration lives in:

- **global** → `~/.specify/stack.md`
- **single-repo** → `./.specify/docs/stack.md`
- **monorepo** → `<repo-root>/.specify/docs/stack.md`

### Effective precedence

1. `~/.specify/stack.md` provides **global user defaults**;
2. `./.specify/docs/stack.md` overrides the global file for a single repository;
3. `<repo-root>/.specify/docs/stack.md` is the canonical repository source in monorepos;
4. package-level `stack.md` files remain **derived views** and do not override the root file.

If `scope=global` is requested, or no repository-local `stack.md` exists, the skill must resolve to `~/.specify/stack.md`.

## Canonical `stack.md` Format

The managed sections must be explicit and stable:

```md
---
updated_at: 2026-04-05T00:00:00Z
---

# Detected Stack

## Active References
- `references/practices/hexagonal-architecture.md`
- `references/frameworks/react.md` — scope: `packages/app`

## Disabled References
- `references/frameworks/langgraph.md`

## Active Skills
- `docs-sync`
- `stack-list`

## Disabled Skills
- `engineer-review` — scope: `packages/legacy-app`

## Active Agents
- `reviewer`
- `backend-architect` — scope: `packages/functions`

## Disabled Agents
- `langgraph-architect`
```

### Writing rules

1. Each artifact appears at most once per effective scope.
2. An artifact cannot be present in both `Active` and `Disabled` for the same type/scope.
3. Items are written as Markdown bullets using the canonical identifier in backticks.
4. Optional scope metadata is written inline using the pattern:

   ```md
   - `<id>` — scope: `<package-path>`
   ```

5. Missing disabled sections must be created when needed.
6. Missing active sections must be created when needed.

## Skill Behavior Contract

### `stack-enable`

`stack-enable` must:

1. resolve the canonical `stack.md` file;
2. normalize the requested `type` and `artifact`;
3. remove the artifact from the matching `Disabled` section for the same scope, if present;
4. add the artifact to the matching `Active` section, if not already present;
5. avoid duplicates;
6. report the source file used and the final state.

**Idempotency**: enabling an already-active artifact must not create duplicate entries and must return a no-op style confirmation.

### `stack-disable`

`stack-disable` must:

1. resolve the canonical `stack.md` file;
2. normalize the requested `type` and `artifact`;
3. remove the artifact from the matching `Active` section for the same scope, if present;
4. add the artifact to the matching `Disabled` section, if not already present;
5. avoid duplicates;
6. report the source file used and the final state.

**Idempotency**: disabling an already-disabled artifact must not create duplicate entries and must return a no-op style confirmation.

### `stack-list`

`stack-list` must:

1. resolve the canonical `stack.md` file;
2. parse all declared `Active` and `Disabled` sections;
3. apply the requested `type`, `state`, and `scope` filters;
4. return grouped results with counts;
5. report the source file used for resolution.

## Default Behavior for Undeclared Artifacts

If an artifact is not declared anywhere in the canonical `stack.md`:

- its default state is **`undeclared/inactive`**;
- it does **not** appear in the default `stack-list` response unless it has been explicitly written to `Active` or `Disabled`;
- `stack-enable` must add it to the relevant `Active` section;
- `stack-disable` must add it to the relevant `Disabled` section, making the disabled intent explicit.

This keeps the format explicit while avoiding hidden defaults.

## Global, Project, and Monorepo Resolution

### Global mode

When `scope=global` is provided, or when no repository-local `stack.md` exists, the skill must:

1. resolve `~/.specify/stack.md`;
2. create it if needed;
3. treat it as the canonical source for **user-level defaults** across projects.

### Repository mode

When running inside a standard single repository:

1. use `./.specify/docs/stack.md` as the canonical project file;
2. let the project file override any overlapping declaration from `~/.specify/stack.md`;
3. fall back to the global file only for defaults that are not declared locally.

### Monorepo mode

A repository must be treated as a monorepo when root signals such as `pnpm-workspace.yaml`, `turbo.json`, or `package.json.workspaces` are present.

In this case:

1. detect the monorepo root;
2. load `<root>/.specify/docs/stack.md` first as the canonical repository file;
3. let the root file override any overlapping declaration from `~/.specify/stack.md`;
4. treat package-level `stack.md` files as informational only;
5. if a package `scope` is provided, write the scoped entry in the **root** `stack.md`, not in the package file.

### Precedence rule

When multiple layers exist, the effective precedence is:

1. explicit `scope=global` → `~/.specify/stack.md`
2. monorepo root → `<root>/.specify/docs/stack.md`
3. single-repo root → `./.specify/docs/stack.md`
4. global defaults from `~/.specify/stack.md`
5. package-level derived views (never authoritative)

When root and package-level `stack.md` files disagree:

- the **root** `stack.md` wins;
- `stack-list` must report the root file as the `source`;
- the skill should mention that a derived package view is out of sync when such a conflict is detected.

## Expected Response Format

### Enable / Disable response

The expected confirmation shape is:

```md
Result: updated
Action: enable
Type: reference
Artifact: `references/frameworks/react.md`
Scope: global | root | `<package-path>`
Source: `~/.specify/stack.md` | `./.specify/docs/stack.md` | `<repo-root>/.specify/docs/stack.md`
State: active
```

If nothing changed, `Result` should be `no-op`.

### List response

The expected list shape is:

```md
Source: `~/.specify/stack.md` | `./.specify/docs/stack.md` | `<repo-root>/.specify/docs/stack.md`
Mode: global | single-repo | monorepo
Filters: `type=skill`, `state=disabled`, `scope=global|packages/app`

## Disabled Skills (2)
- `stack-enable`
- `stack-disable`
```

### List filtering

`stack-list` must support:

- **no filter** → list all declared `references`, `skills`, and `agents`, grouped by `active` and `disabled`;
- **`type` filter** → list only the requested type;
- **`state` filter** → list only `active` or only `disabled` items;
- **combined filter** → e.g. `type=agent` and `state=disabled`.

## Examples

### Example 1 — Global: enable a default reference

**Input**

```text
stack-enable
  type=reference
  artifact=references/practices/hexagonal-architecture.md
  scope=global
```

**Effect**
- writes to `~/.specify/stack.md`
- makes the reference available as a user-level default across projects
- does not depend on a repository-local `stack.md`

### Example 2 — Single-repo: enable a project reference

**Input**

```text
stack-enable
  type=reference
  artifact=references/frameworks/react.md
```

**Effect**
- writes to `./.specify/docs/stack.md`
- ensures the reference is under `## Active References`
- overrides the same artifact if it was only declared globally

### Example 3 — Single-repo: list disabled agents

**Input**

```text
stack-list
  type=agent
  state=disabled
```

**Expected output**
- uses `./.specify/docs/stack.md` as `source`
- returns only `## Disabled Agents`

### Example 4 — Monorepo: disable a package-scoped agent

**Input**

```text
stack-disable
  type=agent
  artifact=langgraph-architect
  scope=packages/functions
```

**Effect**
- detects the monorepo root
- writes the scoped disabled entry in `<root>/.specify/docs/stack.md`
- does not treat `packages/functions/.specify/docs/stack.md` as the source of truth

### Example 5 — Monorepo: list only disabled skills for one package

**Input**

```text
stack-list
  type=skill
  state=disabled
  scope=packages/app
```

**Expected output**
- reads the root `stack.md`
- filters only disabled `skills` entries matching `scope=packages/app`
- reports the root file as the resolved `source`

## Acceptance Criteria

- [x] A clear spec exists for `enable`, `disable`, and `list` skills
- [x] `stack.md` is the source of truth
- [x] Global user defaults are centralized in `~/.specify/stack.md`
- [x] In monorepos, the parent/root `stack.md` is prioritized
- [x] Actions over `references`, `skills`, and `agents` are documented
- [x] Examples and precedence rules are included
- [x] List responses support filtering by type

## Implementation Notes

- Create canonical skills in:
  - `skills/stack-enable/SKILL.md`
  - `skills/stack-disable/SKILL.md`
  - `skills/stack-list/SKILL.md`
- Expose them through thin Copilot adapters in `.github/skills/`
- Keep the behavior definition in the canonical skill files and use this spec as the contract reference
