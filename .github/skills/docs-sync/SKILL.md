---
name: docs-sync
description: 'Detect the current project structure and synchronize `./.specify/docs` from real specs and code. Use after shipping changes, making architectural decisions, or auditing documentation.'
argument-hint: '[scope: init, index, architecture, integrations, or all]'
user-invocable: true
---

# Docs Sync

GitHub Copilot adapter for [`../../../skills/docs-sync/SKILL.md`](../../../skills/docs-sync/SKILL.md).

## When to use

- Updating derived docs after code or architecture changes
- Initializing `./.specify/docs` for a repository
- Auditing docs for drift or missing sections

## Procedure

1. Detect stack and repository structure from real files.
2. Load the relevant knowledge and agent mappings.
3. Update the requested `./.specify/docs/*` files with evidence-based content only.
4. Report updated files, verified sections, skips, and gaps.

## References

- [Source workflow](../../../skills/docs-sync/SKILL.md)
- [Documentation derivation knowledge](../../../knowledge/practices/documentation-derivation.md)
