---
updated_at: 2026-04-05T00:00:00Z
---

# Tasks: Stack-Based AI Artifact Management

## Phase 1 — Canonical Spec and References

### Task 1.1 — Create the feature spec

**Files**
- `.specify/specs/stack-artifact-management/spec.md`

**What**
- Define the contract for `stack-enable`, `stack-disable`, and `stack-list`
- Document the canonical `stack.md` format
- Define `~/.specify/stack.md` as the global user-level default location
- Define root-first precedence in monorepos and local-over-global override rules
- Define default behavior for undeclared artifacts
- Include global, single-repo, and monorepo examples

**Acceptance**
- [ ] The spec covers `references`, `skills`, and `agents`
- [ ] List responses support filtering by type
- [ ] The root `stack.md` rule is explicit
- [ ] The global `~/.specify/stack.md` rule is explicit

### Task 1.2 — Update monorepo guidance

**Files**
- `references/practices/practices/monorepo.md`

**What**
- Document that `<root>/.specify/docs/stack.md` is the source of truth for active/disabled AI artifacts
- Clarify that package-level `stack.md` files are derived views only

**Acceptance**
- [ ] Root-first precedence is documented in the monorepo reference

---

## Phase 2 — Canonical Skills

### Task 2.1 — Create `stack-enable`

**Files**
- `skills/stack-enable/SKILL.md`

**What**
- Accept `type`, `artifact`, and optional `scope`
- Resolve the canonical `stack.md`
- Move the artifact into the matching `Active` section
- Keep the operation idempotent

**Acceptance**
- [ ] The skill documents the enable flow for `reference`, `skill`, and `agent`
- [ ] The skill explicitly writes to the root `stack.md` in monorepos

### Task 2.2 — Create `stack-disable`

**Files**
- `skills/stack-disable/SKILL.md`

**What**
- Accept `type`, `artifact`, and optional `scope`
- Resolve the canonical `stack.md`
- Move the artifact into the matching `Disabled` section
- Keep the operation idempotent

**Acceptance**
- [ ] The skill documents the disable flow for `reference`, `skill`, and `agent`
- [ ] The skill explicitly writes to the root `stack.md` in monorepos

### Task 2.3 — Create `stack-list`

**Files**
- `skills/stack-list/SKILL.md`

**What**
- Accept optional `type`, `state`, and `scope`
- Read the canonical `stack.md`
- Return grouped results with counts and the resolved `source`

**Acceptance**
- [ ] The skill supports `active`, `disabled`, and `all`
- [ ] The skill supports filtering by `reference`, `skill`, or `agent`

---

## Phase 3 — Thin Copilot Adapters

### Task 3.1 — Add wrappers in `.github/skills`

**Files**
- `.github/skills/stack-enable/SKILL.md`
- `.github/skills/stack-disable/SKILL.md`
- `.github/skills/stack-list/SKILL.md`

**What**
- Create thin adapters that point back to the canonical skill files
- Keep descriptions keyword-rich and user-invocable

**Acceptance**
- [ ] Each wrapper references the canonical skill
- [ ] No duplicated behavior prose exists in the wrappers

---

## Phase 4 — Follow-up Validation

### Task 4.1 — Verify examples against real `stack.md` files

**Files to inspect**
- `.specify/docs/stack.md`
- consumer monorepo root and package `stack.md` files

**What**
- Confirm the examples match the current repository conventions
- Check that root-first precedence is consistent with the monorepo reference

**Acceptance**
- [ ] The spec matches the existing root/package `stack.md` layout
- [ ] The examples remain consistent with `references`, not `knowledge`
