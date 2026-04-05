---
updated_at: 2026-04-05T00:00:00Z
---

# Operations — specify-sde

> Workspace customization setup, update, and maintenance procedures.

---

## Lifecycle

`specify-sde` has no runtime process to deploy. Operational work consists of maintaining the Markdown artifacts, syncing documentation, and validating behavior in the supported agent runtimes.

---

## Supported Runtimes

| Runtime | Entry point | Notes |
|---|---|---|
| **GitHub Copilot** | `.github/copilot-instructions.md`, `.github/agents/`, `.github/skills/` | Recommended workspace adapter surface |
| **Claude Code** | `plugin.json`, `agents/` | Compatibility layer preserved during transition |

---

## GitHub Copilot Setup

1. Open this repository in VS Code with **GitHub Copilot Chat** enabled.
2. Keep the workspace customization files under `.github/` versioned with the repository.
3. Use the agent picker for the custom agents and `/` commands for the skill adapters.
4. If discovery appears stale after changes, reload the VS Code window or restart the chat session.

---

## Claude Code Setup

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

```text
❯ specify-sde@local
  Version: 0.1.0
  Scope: user
  Status: ✔ enabled
```

Restart Claude Code after installation for skills and agents to take effect.

---

## Update After Source Changes

When changing the toolkit itself:

1. Update shared rules in `.github/copilot-instructions.md` if the change is global.
2. Update `references/` or `skills/` when the source of truth changes.
3. Keep `.github/` and `agents/` aligned so both runtimes keep the same intent.
4. Refresh `./.specify/docs/` when architecture or usage changes.
5. If Claude compatibility is in use, reinstall the local plugin cache:

```bash
claude plugin uninstall specify-sde@local && claude plugin install specify-sde@local
```

---

## Configuration

No environment variables. No secrets. No runtime config files are required.

---

## Versioning

Version is declared in `plugin.json`. Increment it when there are breaking changes to workflow interfaces, adapter naming, or documented expectations.

---

## Known Constraints

- GitHub Copilot discovery depends on clear `description` metadata and correct folder names under `.github/`.
- Claude Code caches plugin content at install time, so changes may require reinstall.
- The toolkit writes artifacts into the **consuming project's** `./.specify/` directory rather than into this repository at runtime.
