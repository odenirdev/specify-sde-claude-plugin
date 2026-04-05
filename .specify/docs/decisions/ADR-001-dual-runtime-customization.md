---
updated_at: 2026-04-05T00:00:00Z
status: accepted
---

# ADR-001 — Shared Core with Dual Runtime Adapters

## Context

`specify-sde` started as a Claude-oriented Markdown toolkit. As the repository evolved, the same workflows and references needed to be reused by **GitHub Copilot** without copying large blocks of content into multiple runtime-specific files.

The main problems were:

- shared rules appearing in several agents and workflow descriptions;
- runtime-specific entrypoints mixing source-of-truth behavior with adapter metadata;
- documentation describing the toolkit as Claude-only even though the content is reusable.

## Decision

Keep the repository organized as a **shared core + thin runtime adapters** system with explicit canonical ownership:

- `./.specify/docs/` is the primary engineering context and the source of truth for architecture, operations, and ADRs;
- `.github/copilot-instructions.md` holds shared governance for both runtimes;
- `references/` remains the runtime-agnostic engineering references base;
- `skills/` remains the source of truth for reusable workflows;
- `agents/*.md` holds the full role-specific agent guidance;
- `.github/skills/*/SKILL.md` and `.github/agents/*.agent.md` stay intentionally thin and link back to the canonical sources above;
- `README.md` and `CLAUDE.md` remain derived entrypoints subordinate to `./.specify/docs/`.

When the same behavior could live in more than one place, update the canonical layer first and reference it elsewhere instead of copying prose.

## Consequences

### Positive

- shared guidance has a clearer single source of truth by content type;
- Copilot and Claude stay aligned with less maintenance overhead;
- the repository follows a clearer Clean / Hexagonal ordering: `references -> skills -> adapters`;
- future extensions can add new adapters without rewriting the shared core.

### Trade-offs

- maintainers must keep `.github/` and `agents/` aligned while dual support remains;
- only minimal wrapper content should be duplicated across runtimes;
- a lightweight drift validation step is still needed when adapters, docs, or skill names change.

## Follow-up

- validate the minimal GitHub Copilot flow in a real consuming workspace;
- use `docs-sync` plus the documented repo-wide search checklist when removing or renaming skills, references, or adapters;
- keep `README.md`, `CLAUDE.md`, and `./.specify/docs/` aligned whenever the adapter model or entrypoint convention changes.
