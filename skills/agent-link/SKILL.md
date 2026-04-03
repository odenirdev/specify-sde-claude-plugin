---
name: agent-link
description: Audits and updates the "Skills Used" and "Knowledge to Prioritize" sections in one or all agent files under agents/. Triggered when the user wants to sync agent references after adding new skills or knowledge files, review which skills an agent uses, or update a specific agent's pointers.
argument-hint: "[agent-name | all]"
allowed-tools: Read, Glob, Grep, Write, Edit
---

# Agent Link

## Objective

Keep the "Skills Used" and "Knowledge to Prioritize" sections of each `agents/*.md` accurate and complete. Read each agent's purpose, scan existing skills and knowledge files, determine relevance, and update the sections in-place.

---

## When to Use

Use this skill when the user wants to:
- Sync agent references after adding new skills via `knowledge-update` or creating new skill files
- Update a specific agent's "Skills Used" or "Knowledge to Prioritize" sections
- Audit all agents for missing or stale skill/knowledge links

---

## Inputs

- **Agent name** (e.g., `reviewer`) or `all` to process every agent
- Existing `agents/*.md` files
- All files under `skills/` and `knowledge/`

---

## Responsibilities

### Phase 0 — Mode Detection

Check if `plugin.json` exists in the current working directory.

- **Plugin mode**: `plugin.json` found → proceed to Phase 1 (agent file audit).
- **Project mode**: `plugin.json` not found → skip to Phase P1 (project Active Agents update).

---

### PLUGIN MODE

### Phase 1 — Collect Available Skills

1. Glob all files matching `skills/*/SKILL.md`.
2. For each file, read its frontmatter `name` and `description` fields.
3. Build an index:

```
skills/engineer-define/SKILL.md       → engineer-define     — feature definition, scope, approach, risks
skills/engineer-define-spec/SKILL.md  → engineer-define-spec — spec generation, PRD, requirements
skills/engineer-define-tasks/SKILL.md → engineer-define-tasks — task breakdown, phases, acceptance criteria
skills/engineer-review/SKILL.md       → engineer-review      — code review, production risk, architecture alignment
skills/engineer-debug/SKILL.md        → engineer-debug       — root cause analysis, failure investigation
skills/engineer-tradeoff/SKILL.md     → engineer-tradeoff    — option comparison, trade-off analysis
skills/engineer-delivery/SKILL.md     → engineer-delivery    — delivery tracking, progress, blockers
skills/docs-sync/SKILL.md             → docs-sync            — documentation sync, stack detection, agent mapping
skills/knowledge-update/SKILL.md      → knowledge-update     — creating and updating knowledge files
skills/knowledge-link/SKILL.md        → knowledge-link       — syncing knowledge references in skills
skills/agent-link/SKILL.md            → agent-link           — syncing skill/knowledge references in agents
skills/engineer-discovery/SKILL.md    → engineer-discovery   — codebase exploration, context gathering
```

### Phase 2 — Collect Available Knowledge

1. Glob all files matching `knowledge/**/*.md`.
2. For each file, read its first heading and first section line to extract what it covers.
3. Build a knowledge index mirroring what `knowledge-link` uses (see `skills/knowledge-link/SKILL.md` Phase 1).

### Phase 3 — Resolve Target Agents

- If argument is a specific agent name (e.g., `reviewer`): process only `agents/reviewer.md`.
- If argument is `all` or omitted: glob `agents/*.md` and process each one.

### Phase 4 — Analyze Each Agent

For each target agent:

1. Read the full agent file.
2. Extract the agent's **Role**, **Mission**, and **When to Use** sections to understand what it does.
3. For each skill in the index, assess relevance:
   - **Primary**: the skill directly enables the agent's main capability (e.g., `engineer-review` is primary for the reviewer agent)
   - **Secondary**: the skill supports the agent in specific sub-tasks but is not its core capability
   - **Not relevant**: no functional overlap between agent purpose and skill domain

4. For each knowledge file in the index, assess relevance:
   - **Always relevant**: knowledge that applies across all projects for this agent's role (e.g., `error-handling.md` for a reviewer)
   - **Stack-conditional**: knowledge relevant only when the codebase uses a specific technology — add note `(load if detected in stack)`
   - **Not relevant**: no overlap

5. Build updated sections:
   - `## Skills Used`: list primary first, then secondary — one entry per line with a short note
   - `## Knowledge to Prioritize`: list always-relevant first, then conditional with the `(load if detected in stack)` note

### Phase 5 — Update the Sections

1. Locate the existing `## Skills Used` section in the agent file and replace its content.
2. Locate the existing `## Knowledge to Prioritize` section and replace its content.
3. Format:

```markdown
## Skills Used

- `engineer-review` — primary review capability
- `engineer-tradeoff` — when a decision in the code needs evaluation

## Knowledge to Prioritize

- `knowledge/utilities/error-handling.md` — common error handling pitfalls
- `knowledge/utilities/testing.md` — test coverage expectations
- `knowledge/practices/hexagonal-architecture.md` — architecture rules
- `knowledge/frameworks/nestjs.md` — NestJS patterns (load if detected in stack)
- `knowledge/languages/typescript.md` — TypeScript patterns (load if detected in stack)
```

4. If either section does not exist in the agent file, add it before `## Constraints` or at the end of the file.
5. Do NOT modify any other section of the agent file.

### Phase 6 — Report

After processing all target agents, output a summary table:

```
| Agent            | Skills Added | Skills Removed | Knowledge Added | Knowledge Removed |
|---|---|---|---|---|
| reviewer         | 0            | 0              | 1               | 0                 |
| debugger         | 1            | 0              | 0               | 1                 |
```

---

### PROJECT MODE

### Phase P1 — Stack Detection

Inspect the project root for stack signals:

| File | Signal |
|---|---|
| `package.json` | Read `dependencies` and `devDependencies` |
| `go.mod` | Go project |
| `requirements.txt` / `pyproject.toml` | Python project |
| `pom.xml` / `build.gradle` | Java project |

Extract detected technologies (languages, frameworks, libs).

### Phase P2 — Load Agent Map

Read `skills/docs-sync/references/agent-map.md` from the plugin directory (two levels up from `skills/agent-link/`).

Map each detected technology to its relevant agents using the table in `agent-map.md`.

### Phase P3 — Update Active Agents in stack.md

1. Read `./.specify/docs/stack.md`. If the file does not exist, stop and report: "`./.specify/docs/stack.md` not found — run `docs-sync init` first."
2. Locate the `## Active Agents` section.
3. Replace its content with the matched agent identifiers. Format:

```markdown
## Active Agents

- specify-sde:backend-architect
- specify-sde:reviewer
- specify-sde:debugger
- specify-sde:task-planner
- specify-sde:docs-maintainer
```

4. Do NOT modify any other section of `stack.md`.
5. Do NOT add agents that are not listed in `agent-map.md` for the detected stack.

### Phase P4 — Report

Output a summary:

```
## Active Agents Updated

Stack detected: TypeScript, NestJS
Agents linked: 5
File updated: ./.specify/docs/stack.md
```

---

## Output Location

- **Plugin mode**: in-place edit of `agents/<name>.md` — only the `## Skills Used` and `## Knowledge to Prioritize` sections.
- **Project mode**: in-place edit of `./.specify/docs/stack.md` — only the `## Active Agents` section.

---

## Quality Bar

**Plugin mode:**
- [ ] No skill listed that does not exist at `skills/<name>/SKILL.md`
- [ ] No knowledge file listed that does not exist at the path shown
- [ ] Conditional knowledge references are marked `(load if detected in stack)`
- [ ] Primary skills are listed before secondary skills
- [ ] Universal knowledge (practices, utilities) is always included for applicable agents
- [ ] No other section of the agent file was modified
- [ ] Skill names use the short form (e.g., `engineer-review`, not the full path)
- [ ] Knowledge paths use the form `knowledge/<category>/<name>.md` — no absolute paths

**Project mode:**
- [ ] Mode was correctly detected (no `plugin.json` in project root)
- [ ] Stack was detected from actual project files — not assumed
- [ ] Every listed agent was verified to exist in `agent-map.md` for the detected stack
- [ ] Only `## Active Agents` was modified in `stack.md`
- [ ] Agent identifiers use the `specify-sde:<name>` form

---

## Guardrails

- Do NOT rewrite or refactor any other section of the agent file or `stack.md`
- Do NOT add skills that do not exist in `skills/*/SKILL.md`
- Do NOT add knowledge files that do not exist on disk
- Do NOT remove a reference without a clear reason (e.g., skill deleted or zero functional overlap)
- Do NOT create new skills or knowledge files — use `knowledge-update` for knowledge, create skill files manually
- In project mode, do NOT run the plugin-mode phases — they are mutually exclusive
- In project mode, if `./.specify/docs/stack.md` is missing, stop and instruct the user to run `docs-sync init` first
