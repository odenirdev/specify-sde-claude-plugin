# Agent Authoring — Engineering Reference

## Core Principle

An agent is a persona with a defined role, a bounded scope, and a declared toolset. One agent = one responsibility. An agent that does everything is an agent that does nothing well.

## Agent Profile Structure

Agents are defined in Markdown files with YAML frontmatter:

```markdown
---
name: agent-name
description: One-line description of the agent's role and activation signal.
tools:
  - Read
  - Glob
  - Grep
  - Bash
model: sonnet   # optional: haiku | sonnet | opus
---

# Agent Name

You are a [role] specialized in [domain].

## Responsibilities
- What the agent does
- What the agent decides
- What the agent produces

## Behavior
Specific behavioral instructions: how it reasons, what it prioritizes,
what tradeoffs it makes.

## Constraints
- What the agent must NOT do
- When to escalate to the user
- What falls outside scope

## Output
The format and shape of what the agent returns.
```

---

## Runtime Paths

| Runtime | Agent file location |
|---|---|
| **GitHub Copilot** | `.github/agents/<name>.agent.md` |
| **Claude Code** | `agents/<name>.md` (declared in `plugin.json`) |

Keep agent logic in the shared core (`agents/<name>.md`) and reference it from runtime adapters. Never duplicate the persona and instructions across runtimes.

---

## Scope Design

### Single Responsibility
Each agent owns one domain:
- `reviewer` — code correctness, production risk, architecture alignment
- `debugger` — root cause analysis, failure investigation
- `docs-maintainer` — documentation accuracy and currency
- `task-planner` — spec-to-task breakdown and delivery planning

When an agent starts needing a second unrelated capability, split it.

### Tool Declaration
Only declare tools the agent actually uses. Excess tool grants confuse the model about what actions are available and inflate context.

| Tool | When to include |
|---|---|
| `Read`, `Glob`, `Grep` | Any agent that reads the codebase |
| `Bash` | Only when shell execution is required |
| `Write`, `Edit` | Only when the agent produces file output |
| `WebSearch`, `WebFetch` | Only when external data is needed |

---

## Writing the Persona

- State the role in the first sentence: "You are a senior backend engineer..."
- Ground expertise in concrete domain knowledge, not generic intelligence
- Define what the agent prioritizes when tradeoffs arise
- Be prescriptive: "always flag X", "never modify Y without user confirmation"

Example — weak vs. strong:
```
Weak:   "You are a helpful code reviewer."
Strong: "You are a senior backend engineer reviewing a pull request for production risk.
         Prioritize: security vulnerabilities, data integrity issues, missing error handling.
         Flag breaking changes, performance regressions, and missing tests.
         Do not comment on style or formatting — those belong to the linter."
```

---

## Guardrails

Every agent must define what it will not do:

- Destructive operations (file deletion, database drops) → require explicit user confirmation
- Actions visible to others (git push, PR creation, Slack messages) → require explicit confirmation
- Scope creep → the agent must refuse requests outside its domain and redirect

State guardrails with prescriptive language:
```
Never write to disk without explicit user confirmation.
If the task falls outside [domain], state scope and redirect to the appropriate agent.
```

---

## Model Selection

| Model | When to use |
|---|---|
| `haiku` | Lookups, summaries, trivial single-file tasks |
| `sonnet` | Features, moderate-context analysis, standard reviews |
| `opus` | Architecture decisions, critical debugging, deep multi-layer analysis |

Default to `sonnet` unless the task is clearly trivial (haiku) or requires the highest reasoning depth (opus).

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Agent with no tool constraints | Unclear capabilities; model over-reaches |
| Two unrelated responsibilities in one agent | Both suffer; split into two agents |
| Vague persona ("helpful assistant") | No domain grounding; generic output |
| No guardrails section | Agent takes risky actions by default |
| Duplicate logic across runtime adapters | Drift; fix in one place, break in the other |
| Agent description is too broad | Triggers for wrong tasks; costs tokens |

---

## Review Criteria

- [ ] Agent has exactly one domain of responsibility
- [ ] `description` is specific enough to distinguish from other agents
- [ ] Persona is grounded in a concrete role (not "helpful assistant")
- [ ] Tool list includes only tools the agent actually uses
- [ ] Guardrails cover destructive, public, and out-of-scope actions
- [ ] Model is appropriate for the task complexity
- [ ] Agent logic lives in shared core — adapter is a thin wrapper only
- [ ] Output format is defined and verifiable
