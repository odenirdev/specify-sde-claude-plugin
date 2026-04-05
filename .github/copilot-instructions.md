# specify-sde — GitHub Copilot Instructions

This repository is a reusable customization base for **GitHub Copilot** and **Claude Code**.
Edit the canonical layer for the type of change you are making and keep runtime adapters thin.

## Canonical ownership

- `./.specify/docs/index.md` = primary engineering context and navigation
- `.github/copilot-instructions.md` = shared governance for both runtimes
- `references/` = shared engineering references
- `skills/*/SKILL.md` = workflow source of truth
- `agents/*.md` = full role-specific agent guidance
- `.github/skills/*/SKILL.md` and `.github/agents/*.agent.md` = thin Copilot wrappers
- `README.md` and `CLAUDE.md` = derived entrypoints

## Working model

1. Start with `./.specify/docs/index.md` for the current engineering context; use `README.md` for quick orientation and this file for governance.
2. Update `references/`, `skills/`, or `agents/` before touching `.github/` wrappers whenever the source behavior changes.
3. Prefer **referencing** existing `references/` and `skills/` content instead of copying it into new agents or skills.
4. When a rule is shared across multiple agents or skills, centralize it here or in `references/`, not in repeated blocks.

## Shared rules

- Verify claims against code, specs, or file contents before stating them as facts.
- Documentation is **derived** from `./.specify/specs/` or repository files; never speculate.
- Follow **Clean / Hexagonal Architecture** ordering: `domain -> use cases -> adapters`.
- Reuse existing patterns before introducing new abstractions, folders, or workflows.
- Keep agents and skills **single-purpose**, keyword-rich, and easy to discover.
- Keep edits minimal, explicit, and maintainable.

## Context navigation

For the complete architecture and document map, see `./.specify/docs/index.md`. Use this file for governance and maintenance rules, not as a second architecture document.

- **Primary engineering context**: `./.specify/docs/index.md`
- **Shared core**: `references/**/*.md` and `skills/*/SKILL.md`
- **Runtime adapters**: `.github/agents/*.agent.md`, `.github/skills/*/SKILL.md`, `agents/*.md`, and `plugin.json`
- **Verification source material**: `./.specify/specs/**/*` and repository files

## Maintenance checklist

- If you change shared rules, update this file first.
- If you add or rename a skill, keep `skills/` and `.github/skills/` aligned.
- If you add or change an agent, update the canonical file in `agents/` first and then keep the Copilot wrapper aligned.
- If you delete a skill, reference, or adapter, remove stale links from docs and wrappers in the same change.
- If the architecture or entrypoint convention changes, update `./.specify/docs/` first, then sync `README.md` and `CLAUDE.md`.
