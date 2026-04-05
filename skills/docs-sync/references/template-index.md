# [Project Name]

> One paragraph describing what this system does, who uses it, and why it exists. Be concrete — "a B2B SaaS platform that manages X for Y" is better than "a modern scalable system."

---

## Stack & Architecture

| Layer | Technology | Version | Notes |
|---|---|---|---|
| Runtime | Node.js / Go | vX.X | |
| Framework | NestJS / Gin | vX.X | |
| Persistence | PostgreSQL | vX.X | |
| ORM | Prisma / sqlc | vX.X | |
| Queue | [if applicable] | | |
| Cache | [if applicable] | | |
| Frontend | React / Next.js | vX.X | |

**Architecture pattern**: [e.g., Hexagonal / Modular Monolith / Microservices]

See [architecture.md](./architecture.md) for full system design and ADRs.

---

## Integrations

| Service | Purpose | Auth |
|---|---|---|
| [Service Name] | [What it does for us] | [API key / OAuth / mTLS] |

See [integrations.md](./integrations.md) for contracts and failure behaviors.

---

## Main Features

- [Feature 1]: [One sentence description]
- [Feature 2]: [One sentence description]
- [Feature 3]: [One sentence description]
- [Feature N]: ...

---

## Domain

Key domain concepts:

- **[Concept]**: [Definition — what it means in this system's context]
- **[Concept]**: [Definition]
- **[Concept]**: [Definition]

---

## Derived Documentation

| Document | Description |
|---|---|
| [stack.md](./stack.md) | Detected stack, active agents, active references |
| [architecture.md](./architecture.md) | System design, boundaries, component map |
| [integrations.md](./integrations.md) | External service contracts and failure behavior |
| [operations.md](./operations.md) | Deploy procedures, config, runbooks |
| [decisions/](./decisions/) | Architectural Decision Records (ADRs) |

---

## Operational Links

| Resource | Link |
|---|---|
| Production dashboard | [URL or description] |
| Error tracking | [Sentry / Datadog / etc.] |
| CI/CD pipeline | [GitHub Actions / etc.] |
| Deployment runbook | [Link to operations.md or external] |

---

## Architectural Decisions

| ADR | Decision | Date |
|---|---|---|
| [ADR-001](./decisions/ADR-001-[name].md) | [One-line summary] | YYYY-MM-DD |

---

## Hypotheses & Pending Items

> Unresolved questions and unvalidated assumptions. Remove items when they are confirmed or refuted.

- [ ] [Hypothesis or open question]
- [ ] [Known unknown]
