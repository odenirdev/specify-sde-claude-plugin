---
name: stack-init
description: Initialize `.specify/stack.yml` for a project — the declarative YAML source of truth for AI artifact management. Migrates from `stack.md` when present, or creates a minimal config from scratch.
argument-hint: "[--migrate] [--force]"
user-invocable: true
---

# Stack Init

GitHub Copilot adapter for [`../../../skills/stack-init/SKILL.md`](../../../skills/stack-init/SKILL.md).

Use this skill to create `.specify/stack.yml`, establishing it as the primary source of truth consumed by `stack-enable`, `stack-disable`, and `stack-list`. Migrates from an existing `stack.md` when one is found.
