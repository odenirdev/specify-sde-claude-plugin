# Prompt Engineering — Engineering Reference

## Core Principle

A prompt is a specification for agent behavior. The quality of output directly correlates with the precision, structure, and context of the instruction. More words do not equal a better prompt — specificity does.

## Anatomy of an Effective Instruction

Every instruction to an AI agent should establish:

| Element | Question it answers |
|---|---|
| **Role** | Who is the agent in this context? |
| **Objective** | What must be produced or decided? |
| **Constraints** | What is out of scope or forbidden? |
| **Format** | What does the output look like? |
| **Tone/style** | How should the agent communicate? |

Example — weak vs. strong:
```
Weak:   "Review the code."
Strong: "Review the diff below as a senior backend engineer. Flag security issues,
         non-idiomatic patterns, and missing error handling. Output a bullet list
         grouped by severity (critical / warning / note). Skip style nitpicks."
```

---

## Core Techniques

### Zero-Shot
Use for tasks that are clear, bounded, and self-contained. No examples needed.
```
Summarize the following ADR in three bullet points. Output only the bullets.
```

### Few-Shot
Use when output format or pattern matters. Provide 2–5 consistent examples.
```
Classify each item as bug / feature / chore:
- "NPE on null user" → bug
- "Add dark mode" → feature
- "Bump lodash to 4.17.21" → chore
Now classify: "Retry logic on HTTP 429"
```

### Chain-of-Thought
Use for reasoning-heavy tasks. Instruct the agent to think before answering.
```
Before proposing a solution, reason through: what could cause this error,
which causes are most likely given the stack trace, what each fix would change.
Then recommend one fix.
```

### Role-Based Instructions
Grounding the agent in a role narrows the response style and activates domain heuristics:
```
You are a database migration specialist reviewing a schema change on a 50M-row table.
```

---

## Context Engineering

Context window space is finite. Every token competes with conversation history, tool results, and file contents. Manage context intentionally:

- **System prompt**: stable persona, constraints, output format — loaded once
- **User turn**: task-specific inputs — minimize repetition
- **Tool results**: raw data — summarize before passing forward when large
- **Examples**: load only when format is genuinely ambiguous
- **File content**: load only the relevant section, not the whole file

Rule: if removing a context block would not change the output, remove it.

---

## Writing Skills and Agent Instructions

When writing instructions for reusable skills or agents (SKILL.md, agent profiles):

- **Lead with the trigger** — state when the agent should activate before what it does
- **Use prescriptive language** — "always", "never", "must", "do not" instead of "try to" or "consider"
- **Define the output contract** — format, sections, length constraints
- **State guardrails explicitly** — what the agent must refuse or escalate
- **Use headers and tables** — structure is parsed faster than prose
- **Avoid laundry lists** — five precise rules beat twenty vague ones

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| "Do your best" / "Be thorough" | Undefined — agent fills the gap with assumptions |
| Nested ambiguity ("if applicable, maybe…") | Agent ignores the condition |
| Long preamble before the actual ask | Context noise; the task gets buried |
| Format implied, not stated | Output shape varies per run |
| Contradictory rules in same prompt | Agent picks one; you don't know which |
| Stuffing edge cases as a list | Noise; be selective — cover principles, not every case |

---

## Iterative Refinement

Treat prompt development as code development:

1. Write a minimal prompt
2. Run on representative inputs
3. Identify failure modes (wrong format, wrong scope, missing case)
4. Add the smallest possible fix — one rule at a time
5. Retest; confirm the fix does not break prior outputs
6. Repeat

Do not rewrite the entire prompt when one sentence fails. Diagnose before changing.

---

## Review Criteria

- [ ] Role is defined or clearly implied by context
- [ ] Objective is stated in one sentence
- [ ] Output format is explicit (bullets, table, JSON, markdown, etc.)
- [ ] Scope constraints are stated ("only", "do not", "skip")
- [ ] No contradictory rules
- [ ] Examples are present only where format is genuinely ambiguous
- [ ] Guardrails cover the most dangerous failure modes
- [ ] Prompt fits its purpose without unnecessary context
