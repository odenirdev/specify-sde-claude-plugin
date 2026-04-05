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

When changing the toolkit itself, edit the canonical layer first and sync outward:

1. Update `references/**/*.md` when the shared engineering guidance changes.
2. Update `skills/*/SKILL.md` when workflow behavior changes.
3. Update `agents/*.md` when a role's detailed contract or examples change.
4. Keep `.github/skills/*/SKILL.md` and `.github/agents/*.agent.md` thin and aligned with the canonical files above.
5. Update `.github/copilot-instructions.md` only for shared governance changes.
6. Refresh `./.specify/docs/` when architecture, ownership, or maintenance flow changes.
7. If Claude compatibility is in use, reinstall the local plugin cache:

```bash
claude plugin uninstall specify-sde@local && claude plugin install specify-sde@local
```

## Source-of-Truth Maintenance Checklist

Before merging a customization change, verify:

- [ ] The change was made in the canonical layer first, not only in a Copilot wrapper
- [ ] `README.md` and `CLAUDE.md` still defer to `./.specify/docs`
- [ ] `.github/skills/*/SKILL.md` still points to `skills/*/SKILL.md`
- [ ] `.github/agents/*.agent.md` stays link-oriented and points back to `agents/*.md`
- [ ] Deleted skills, references, or adapters were removed from docs and wrappers in the same change

## Drift Checks

Run a quick repository-wide search after any cleanup or deletion:

```bash
rg -n "Source workflow|Legacy .* adapter|Primary engineering context|docs-sync:start" \
  .github agents .specify/docs README.md CLAUDE.md
```

Use the result to confirm that:
- Copilot skill adapters still reference the correct source workflow;
- Copilot agent adapters still link to the detailed role file;
- root entrypoints still point back to the canonical docs;
- no stale references remain after deleting or renaming a skill.

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
