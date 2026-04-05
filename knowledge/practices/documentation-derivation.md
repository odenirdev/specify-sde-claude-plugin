# Documentation Derivation — Engineering Reference

## Core Principle

Documentation must be derived from two sources of truth: specs and code. Documentation that is written independently from either will drift and become misleading. The goal is not to document everything — it's to document what cannot be inferred from reading the code.

## What Belongs in Docs

| Worth documenting | Not worth documenting |
|---|---|
| Why a decision was made | What the code does (readable code is self-documenting) |
| Non-obvious constraints | Obvious flows |
| Integration contracts | Internal implementation details |
| Operational procedures | Standard CRUD operations |
| Architecture decisions (ADRs) | Boilerplate descriptions |
| Domain concepts new engineers won't know | Framework conventions |

## Entry-Point Convention

`./.specify/docs` is the engineering source of truth. The repository may expose other entrypoints, but they are derived and must stay subordinate to the docs.

| File | Role | Rule |
|---|---|---|
| `./.specify/docs/` | Canonical engineering context | Full detail lives here; verify facts against specs and code |
| `CLAUDE.md` | Agent/tool bridge | Keep minimal and link-oriented; never become a second source of truth |
| `README.md` | Human overview and getting started | Keep short, high-signal, and link back to `./.specify/docs/index.md` |

If these files disagree, `./.specify/docs` wins.

## Managed README Blocks

When `docs-sync` updates `README.md`, it should only touch the managed block below:

```markdown
<!-- docs-sync:start -->
...derived overview, getting started, commands, structure, links...
<!-- docs-sync:end -->
```

Rules:
- Preserve all manual content outside the block.
- Keep the block short; link to detailed docs instead of duplicating them.
- If the markers are missing, insert a single managed block rather than rewriting the whole file.
- Never describe future plans as if they were implemented.

## `./.specify/docs` Structure

```
docs/
  index.md              # Entry point — project summary and links
  architecture.md       # System design, boundaries, ADRs
  integrations.md       # External services and contracts
  operations.md         # Deploy, config, runbooks
  domain/               # Domain glossary and concepts
  decisions/            # ADR files (ADR-001-*.md, ADR-002-*.md)
```

## `index.md` is the Anchor

The `index.md` must be kept current. It answers the "what is this system?" question for any new engineer in under 5 minutes.

Sections:
1. **Project Summary** — one paragraph, what it does and why it exists
2. **Stack** — tech, versions, rationale
3. **Architecture** — one-paragraph overview + link to architecture.md
4. **Integrations** — list of external systems + link to integrations.md
5. **Main Features** — 5-10 bullet points of what the system does
6. **Domain** — key domain concepts
7. **Documentation Links** — links to all docs files
8. **Operational Links** — dashboards, runbooks, CI/CD
9. **Architectural Decisions** — list of ADRs with one-line summaries
10. **Hypotheses & Pending Items** — known unknowns, open questions

## ADR Format

Document architectural decisions as they are made. Small decisions that seem obvious become mysteries six months later.

```markdown
# ADR-001: Use Prisma as ORM

**Status**: Accepted
**Date**: 2024-03-15

## Context
We needed an ORM for the Node.js backend. Options considered: TypeORM, Drizzle, Prisma.

## Decision
Use Prisma because: type-safe client generation, migration tooling, Prisma Studio for inspection.

## Consequences
- Schema is the source of truth for types (positive)
- Less SQL expressiveness than Drizzle (accepted trade-off)
- Prisma version upgrades require schema regeneration (manageable)
```

---

## Sync Rules

When updating docs, apply these rules:

1. **Specs first**: If a spec changed, update docs to reflect it. Never invent content not backed by specs or code.
2. **Observe before writing**: Read the relevant code or spec before updating docs. Don't write from memory.
3. **Preserve structure**: Don't reorganize or rename sections without a clear reason. Other tools and team members rely on consistent structure.
4. **No speculation**: Hypotheses belong in the "Hypotheses & Pending Items" section. Confirmed facts belong everywhere else.
5. **Minimal change**: Update only what changed. Broad rewrites introduce errors and lose history.

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Docs written after code, from memory | Inaccurate, stale immediately |
| Docs that describe the code line by line | Maintenance nightmare |
| Missing ADRs | Future engineers repeat solved problems |
| `index.md` not updated when features ship | Entry point becomes misleading |
| `README.md` grows into a second architecture document | Drift and duplication with `./.specify/docs` |
| `CLAUDE.md` becomes a second source of truth | Agents follow stale or conflicting guidance |
| Docs that speculate about future plans | Treated as facts by future readers |

---

## Review Criteria

- [ ] All documented facts are traceable to a spec or code location
- [ ] `index.md` reflects current state of the system
- [ ] ADRs exist for non-obvious architectural decisions
- [ ] `README.md` stays short and uses the managed block convention when docs-sync owns it
- [ ] `CLAUDE.md` stays minimal and points back to `./.specify/docs`
- [ ] No speculative content outside "Hypotheses" section
- [ ] Links in docs point to valid, current resources
- [ ] Outdated sections are either updated or removed
