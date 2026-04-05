---
updated_at: 2026-04-05T00:00:00Z
---

# specify-sde

A modular Software Development Engineering toolkit for **GitHub Copilot** and **Claude Code**. The shared core lives in `knowledge/` and `skills/`, while `.github/` and `agents/` expose that core to different agent runtimes.

---

## Stack & Architecture

| Layer | Technology | Notes |
|---|---|---|
| Runtime | None | Pure Markdown customization toolkit, no build step |
| Formats | `.github` workspace customizations + `plugin.json` compatibility | Supports GitHub Copilot and Claude Code |
| Content | Markdown | Knowledge, skills, agents, docs, and adapter files |

**Architecture pattern**: Shared core + runtime adapters, aligned with Clean / Hexagonal boundaries (`knowledge -> skills -> adapters`).

See [architecture.md](./architecture.md) for the component map and runtime separation.

---

## Main Features

- **Shared knowledge core**: reusable engineering guidance in `knowledge/`
- **Reusable workflows**: `engineer-discovery`, `engineer-define`, `engineer-review`, `docs-sync`, and related skills in `skills/`
- **GitHub Copilot adapters**: workspace instructions, custom agents, and skill wrappers in `.github/`
- **Claude compatibility layer**: existing `agents/*.md` and `plugin.json`
- **Derived documentation**: `./.specify/docs` stays synchronized with the repository architecture and usage model

---

## Domain

Key concepts in this toolkit's model:

- **Spec**: Source of truth for a feature — problem, approach, constraints, and tasks. Lives in `./.specify/specs/<slug>/`.
- **Knowledge**: Shared, runtime-agnostic engineering reference for a technology or practice. Lives in `knowledge/`.
- **Skill**: A reusable workflow defined in `skills/<name>/SKILL.md` and exposed to runtimes through thin adapters.
- **Runtime Adapter**: A Copilot or Claude entrypoint that references the shared core instead of copying it. Lives in `.github/` or `agents/`.
- **Docs**: Derived documentation in `./.specify/docs/`. Never invented — always tied to specs or repository evidence.

---

## Engineering Agents

Primary agent roles configured for this toolkit:

| Agent | Role |
|---|---|
| `reviewer` | Code review, production risk assessment, architecture alignment |
| `backend-architect` | Backend design, persistence, and integration planning |
| `debugger` | Root-cause analysis and failure investigation |
| `task-planner` | Spec-to-task breakdown and delivery planning |
| `docs-maintainer` | Documentation accuracy and currency |
| `langgraph-architect` | AI orchestration design for LangGraph / LangChain workloads |

---

## Derived Documentation

| Document | Description |
|---|---|
| [stack.md](./stack.md) | Detected runtime formats, active agents, active shared knowledge |
| [architecture.md](./architecture.md) | Shared-core / adapter structure and component relationships |
| [operations.md](./operations.md) | Setup, update, and maintenance procedures for both runtimes |
| [decisions/](./decisions/) | Architectural Decision Records (ADRs) |

---

## Hypotheses & Pending Items

> Unresolved questions and unvalidated assumptions. Remove items when confirmed or refuted.

- [ ] Validate the minimal GitHub Copilot flow in a real consuming workspace and capture any discovery quirks as follow-up guidance.
