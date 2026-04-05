# specify-sde

A modular Software Development Engineering toolkit for **GitHub Copilot** and **Claude Code**.

> [`./.specify/docs/index.md`](./.specify/docs/index.md) is the primary engineering context. This `README.md` stays intentionally short and points there for the detailed architecture, operations, and decisions.

<!-- docs-sync:start -->
## Overview

`specify-sde` keeps reusable engineering references in `references/`, workflow skills in `skills/`, and thin runtime adapters in `.github/`, `agents/`, and `plugin.json`. The `docs-sync` workflow keeps `./.specify/docs`, `CLAUDE.md`, and this managed `README.md` block aligned from repository evidence.

## Getting Started

1. Read [`./.specify/docs/index.md`](./.specify/docs/index.md) for the current architecture and operating model.
2. Review [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) for shared workspace rules.
3. Explore [`references/`](./references/) and [`skills/`](./skills/) before changing adapters or docs.
4. Use the relevant agent or workflow (`reviewer`, `debugger`, `docs-sync`, `engineer-define`, etc.).

## Main Workflows

| Workflow | Use when |
|---|---|
| `docs-sync` | Refresh `./.specify/docs`, `CLAUDE.md`, and the managed `README.md` block |
| `engineer-discovery` | Capture requirements into `prd.md` |
| `engineer-define` | Generate `spec.md` and `tasks.md` |
| `engineer-review` | Review diffs, modules, or architecture changes |

## Structure

| Path | Purpose |
|---|---|
| `references/` | Shared engineering references |
| `skills/` | Reusable workflows |
| `.github/` | GitHub Copilot adapters |
| `agents/` | Claude compatibility adapters |
| `./.specify/docs/` | Canonical derived engineering context |

## Detailed Docs

- [`./.specify/docs/index.md`](./.specify/docs/index.md)
- [`./.specify/docs/architecture.md`](./.specify/docs/architecture.md)
- [`./.specify/docs/operations.md`](./.specify/docs/operations.md)
- [`./.specify/docs/decisions/`](./.specify/docs/decisions/)

## Claude Compatibility

```bash
claude plugin marketplace add /Users/odenirgomes/.claude/plugins/marketplaces/local
claude plugin install specify-sde@local
claude plugin list
```
<!-- docs-sync:end -->

## Maintenance note

Keep the managed block derived and high-signal. Detailed rules, architecture, and stack notes belong in `./.specify/docs/` and `.github/copilot-instructions.md`.
