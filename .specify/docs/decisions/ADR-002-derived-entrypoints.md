---
updated_at: 2026-04-05T00:00:00Z
status: accepted
---

# ADR-002 — Derived Repository Entry Points

## Context

`docs-sync` already kept `./.specify/docs/` aligned with repository facts, but it did not guarantee the same alignment for the repository entrypoints used by humans and agent runtimes.

This created three recurring risks:

- `README.md` could drift away from the real architecture and onboarding flow;
- `CLAUDE.md` could accumulate duplicated context that should live in the derived docs;
- new projects using `specify-sde` could start with inconsistent entrypoints even when `./.specify/docs/` was accurate.

## Decision

Keep `./.specify/docs/` as the single source of truth for engineering context and extend the `docs-sync` convention so that:

- root `CLAUDE.md` is created or normalized as a minimal bridge to `./.specify/docs/index.md`;
- in monorepos, package-level `CLAUDE.md` files are created or updated only when the package has local `./.specify/docs/`;
- root `README.md` is kept short and synchronized through one managed block:
  - `<!-- docs-sync:start -->`
  - derived summary content
  - `<!-- docs-sync:end -->`
- any manual content outside the managed README block is preserved.

## Consequences

### Positive

- humans and agent runtimes start from a consistent, low-drift entrypoint layer;
- `README.md` stays concise and link-oriented instead of competing with detailed docs;
- `CLAUDE.md` remains a bridge, not a parallel source of truth;
- monorepo package bridges are only created where local docs actually exist.

### Trade-offs

- maintainers must preserve the README managed block convention;
- package-level `CLAUDE.md` generation depends on correct package docs detection;
- projects with heavily custom READMEs may still need light manual curation outside the managed block.

## Limits

This decision does **not** make `README.md` or `CLAUDE.md` authoritative documentation. Detailed architecture, rules, integrations, and operations continue to belong in `./.specify/docs/`.
