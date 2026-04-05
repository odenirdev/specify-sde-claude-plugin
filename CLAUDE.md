# CLAUDE.md

This file is the **Claude Code context bridge** for this repository. It stays intentionally minimal and routes Claude to the derived docs instead of duplicating engineering guidance.

## Context Order

1. **Primary engineering context**: [`./.specify/docs/index.md`](./.specify/docs/index.md) — current architecture, operating model, and document map.
2. **Rules and governance**: [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) — shared workspace rules and maintenance expectations.
3. **Quick orientation**: [`README.md`](./README.md) — repository overview and adapter layout.
4. **Shared core**: [`references/`](./references/) and [`skills/`](./skills/) — reusable references and workflows.

> Keep this file thin: prefer cross-references to the shared docs and instructions instead of copying rules here. If this file and `./.specify/docs` ever disagree, `./.specify/docs` wins.

---