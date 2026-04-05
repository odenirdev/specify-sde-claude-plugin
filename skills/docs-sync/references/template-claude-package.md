# CLAUDE.md

This file is the package-level context bridge. It should point first to the package docs and then to the monorepo root context when cross-package guidance is needed.

## Context Order

1. **Package engineering context**: [`./.specify/docs/index.md`](./.specify/docs/index.md) — package-specific architecture, stack, and APIs.
2. **Monorepo engineering context**: [`<root-relative-path>/.specify/docs/index.md`](<root-relative-path>/.specify/docs/index.md) — shared architecture and cross-cutting concerns.
3. **Rules and governance**: [`<root-relative-path>/.github/copilot-instructions.md`](<root-relative-path>/.github/copilot-instructions.md) — shared workspace rules. Omit if not present.
4. **Verification sources**: `./src`, tests, and root `./.specify/specs/` — confirm detailed behavior here.

> Keep this file minimal. Do not restate package architecture in full — link to the docs instead.
