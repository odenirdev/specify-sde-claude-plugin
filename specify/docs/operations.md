---
updated_at: 2026-04-03T00:00:00Z
---

# Operations — specify-sde

> Plugin installation, update, and maintenance procedures.

---

## Plugin Lifecycle

This plugin has no runtime to deploy. Operations consist of installation, updates, and reinstallation after source changes.

---

## Installation

### 1. Register the local marketplace (once)

```bash
claude plugin marketplace add /Users/odenirgomes/.claude/plugins/marketplaces/local
```

Skip if the `local` marketplace is already listed in `known_marketplaces.json`.

### 2. Install the plugin

```bash
claude plugin install specify-sde@local
```

### 3. Verify

```bash
claude plugin list
```

Expected output:
```
❯ specify-sde@local
  Version: 0.1.0
  Scope: user
  Status: ✔ enabled
```

Restart Claude Code after installation for skills and agents to take effect.

---

## Update After Source Changes

If files inside this repository are modified, reinstall to sync the plugin cache:

```bash
claude plugin uninstall specify-sde@local && claude plugin install specify-sde@local
```

---

## Configuration

No environment variables. No secrets. No config files required.

---

## Versioning

Version is declared in `plugin.json`. Increment on breaking changes to skill interfaces or output formats.

---

## Known Constraints

- Changes to `SKILL.md` files require plugin reinstall to take effect — Claude Code caches plugin content at install time.
- The plugin operates on the consuming project's `./specify/` directory — it does not create files in its own directory at runtime.
