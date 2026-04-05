---
name: docs-sync
description: 'Detect the current project structure and synchronize `./.specify/docs`, `CLAUDE.md`, and managed `README.md` content from real specs and code. Use after shipping changes, making architectural decisions, or auditing documentation.'
argument-hint: '[scope: init, index, architecture, integrations, claude, readme, or all]'
user-invocable: true
---

# Docs Sync

GitHub Copilot adapter for [`../../../skills/docs-sync/SKILL.md`](../../../skills/docs-sync/SKILL.md).

## When to use

- Updating derived docs after code or architecture changes
- Initializing `./.specify/docs` for a repository
- Refreshing the `CLAUDE.md` bridge or managed `README.md` overview
- Auditing docs for drift or missing sections

## Procedure

1. Detect stack and repository structure from real files.
2. Load the relevant reference and agent mappings.
3. Update the requested `./.specify/docs/*` files plus the derived `CLAUDE.md` and managed `README.md` content with evidence-based data only.
4. Report updated files, verified sections, skips, and gaps.

## References

- [Source workflow](../../../skills/docs-sync/SKILL.md)
- [Documentation derivation reference](../../../references/practices/documentation-derivation.md)
