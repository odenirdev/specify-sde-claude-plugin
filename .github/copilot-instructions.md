# specify-sde — GitHub Copilot Instructions

This repository is a reusable customization base for **GitHub Copilot** and **Claude Code**.
Treat the content as a layered system:

- `knowledge/` = shared engineering knowledge (**domain**)
- `skills/` = reusable workflows (**use cases**)
- `.github/` and `agents/` = runtime adapters (**adapters**)
- `./.specify/docs/` = derived documentation for how the toolkit works today

## Working model

1. Read `README.md` and `./.specify/docs/index.md` before making structural changes.
2. Prefer **referencing** existing `knowledge/` and `skills/` content instead of copying it into new agents or skills.
3. When a rule is shared across multiple agents or skills, centralize it here or in `knowledge/`, not in repeated blocks.

## Shared rules

- Verify claims against code, specs, or file contents before stating them as facts.
- Documentation is **derived** from `./.specify/specs/` or repository files; never speculate.
- Follow **Clean / Hexagonal Architecture** ordering: `domain -> use cases -> adapters`.
- Reuse existing patterns before introducing new abstractions, folders, or workflows.
- Keep agents and skills **single-purpose**, keyword-rich, and easy to discover.
- Keep edits minimal, explicit, and maintainable.

## Source of truth

- Shared knowledge: `knowledge/**/*.md`
- Reusable workflows: `skills/*/SKILL.md`
- Derived docs: `./.specify/docs/*`
- GitHub Copilot adapters: `.github/agents/*.agent.md` and `.github/skills/*/SKILL.md`
- Claude compatibility layer: `agents/*.md` and `plugin.json`

## Maintenance checklist

- If you change shared rules, update this file first.
- If you add or rename a skill, keep `skills/` and `.github/skills/` aligned.
- If you add or change an agent, keep the Copilot adapter and the legacy adapter aligned.
- If the architecture changes, update `README.md` and `./.specify/docs/` in the same change.
