---
updated_at: 2026-04-05T00:00:00Z
status: accepted
---

# ADR-001 — Shared Core with Dual Runtime Adapters

## Context

`specify-sde` started as a Claude-oriented Markdown toolkit. As the repository evolved, the same workflows and knowledge needed to be reused by **GitHub Copilot** without copying large blocks of content into multiple runtime-specific files.

The main problems were:

- shared rules appearing in several agents and workflow descriptions;
- runtime-specific entrypoints mixing source-of-truth behavior with adapter metadata;
- documentation describing the toolkit as Claude-only even though the content is reusable.

## Decision

Keep the repository organized as a **shared core + runtime adapters** system:

- `knowledge/` remains the runtime-agnostic engineering knowledge base;
- `skills/` remains the source of truth for reusable workflows;
- `.github/` becomes the GitHub Copilot adapter surface (`copilot-instructions.md`, custom agents, skill wrappers);
- `agents/` and `plugin.json` remain as the Claude compatibility layer during the transition.

Global rules are centralized in `.github/copilot-instructions.md`, and runtime wrappers should **reference** shared knowledge and workflows instead of copying them.

## Consequences

### Positive

- shared guidance has a single source of truth;
- Copilot and Claude stay aligned with less maintenance overhead;
- the repository follows a clearer Clean / Hexagonal ordering: `knowledge -> skills -> adapters`;
- future extensions can add new adapters without rewriting the shared core.

### Trade-offs

- maintainers must keep `.github/` and `agents/` aligned while dual support remains;
- a minimal runtime validation step is still needed when adapter files change.

## Follow-up

- validate the minimal GitHub Copilot flow in a real consuming workspace;
- keep `README.md` and `./.specify/docs/` aligned whenever the adapter model changes.
