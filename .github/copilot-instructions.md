# specify-sde — GitHub Copilot Instructions

This repository is a reusable customization base for **GitHub Copilot** and **Claude Code**.
Treat the content as a layered system:

- `knowledge/` = shared engineering knowledge (**domain**)
- `skills/` = reusable workflows (**use cases**)
- `.github/` and `agents/` = runtime adapters (**adapters**)
- `./.specify/docs/` = **primary engineering context** for how the toolkit works today (**derived from specs and code**)

## Working model

1. Start with `./.specify/docs/index.md` for the current engineering context; use `README.md` for quick orientation and this file for governance.
2. Prefer **referencing** existing `knowledge/` and `skills/` content instead of copying it into new agents or skills.
3. When a rule is shared across multiple agents or skills, centralize it here or in `knowledge/`, not in repeated blocks.

## Shared rules

- Verify claims against code, specs, or file contents before stating them as facts.
- Documentation is **derived** from `./.specify/specs/` or repository files; never speculate.
- Follow **Clean / Hexagonal Architecture** ordering: `domain -> use cases -> adapters`.
- Reuse existing patterns before introducing new abstractions, folders, or workflows.
- Keep agents and skills **single-purpose**, keyword-rich, and easy to discover.
- Keep edits minimal, explicit, and maintainable.

## Context hierarchy

- **Primary engineering context**: `./.specify/docs/index.md` — current architecture, operating model, and navigation.
- **Rules and governance**: this file — shared maintenance rules for Copilot and Claude adapters.
- **Shared knowledge**: `knowledge/**/*.md`
- **Reusable workflows**: `skills/*/SKILL.md`
- **Verification source material**: `./.specify/specs/**/*` and repository files
- **Runtime adapters**: `.github/agents/*.agent.md`, `.github/skills/*/SKILL.md`, `agents/*.md`, and `plugin.json`

## Maintenance checklist

- If you change shared rules, update this file first.
- If you add or rename a skill, keep `skills/` and `.github/skills/` aligned.
- If you add or change an agent, keep the Copilot adapter and the legacy adapter aligned.
- If the architecture or entrypoint convention changes, update `README.md`, `CLAUDE.md`, and `./.specify/docs/` in the same change.
