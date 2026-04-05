# CLAUDE.md

This file is the minimal context bridge for the repository. It must stay intentionally thin and point to the derived docs instead of duplicating them.

## Context Order

1. **Primary engineering context**: [`./.specify/docs/index.md`](./.specify/docs/index.md) — current architecture, operating model, and document map.
2. **Rules and governance**: [`.github/copilot-instructions.md`](./.github/copilot-instructions.md) — shared workspace rules and maintenance expectations. If the file does not exist, omit this line.
3. **Quick orientation**: [`README.md`](./README.md) — short human overview and getting started.
4. **Verification sources**: `./.specify/specs/` and repository files — confirm detailed behavior here.

> Keep this file minimal. If this file and `./.specify/docs` disagree, `./.specify/docs` wins.
