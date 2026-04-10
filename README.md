# specify-sde

A modular Software Development Engineering toolkit for **GitHub Copilot** and **Claude Code**.

> [`./.specify/docs/index.md`](./.specify/docs/index.md) is the primary engineering context. This `README.md` stays intentionally short and points there for the detailed architecture, operations, and decisions.

<!-- docs-sync:start -->
## Overview

`specify-sde` keeps reusable engineering references in `references/`, utility skills in `skills/`, and thin runtime adapters in `.github/`, `agents/`, and `plugin.json`. Engineering workflow skills (discovery, define, delivery, debug, review, docs-sync) live in the companion plugin [`specify-engineer`](https://github.com/ogs-tech/specify-engineer).

## Getting Started

1. Read [`./.specify/docs/index.md`](./.specify/docs/index.md) for the current architecture and operating model.
2. Review [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) for shared workspace rules.
3. Explore [`references/`](./references/) and [`skills/`](./skills/) before changing adapters or docs.
4. Use the relevant agent (`reviewer`, `debugger`, `backend-architect`, etc.) or skill (`review-triage`, `stack-list`, etc.).

## Main Skills

| Skill | Use when |
|---|---|
| `review-triage` | Triage and prioritize review findings |
| `review-topic-triage` | Triage review findings by topic |
| `stack-list` | List active references and adapters |
| `stack-enable` | Enable a reference or adapter |
| `stack-disable` | Disable a reference or adapter |

## Structure

| Path | Purpose |
|---|---|
| `references/` | Shared engineering references |
| `skills/` | Utility workflow skills |
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
