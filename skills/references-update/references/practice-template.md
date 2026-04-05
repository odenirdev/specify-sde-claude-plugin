# [PracticeName] — Engineering Reference

> Engineering practices are intentional patterns applied consistently across a codebase or team — architecture styles, design approaches, workflow standards.

## Best Practices

### [Core Principle — e.g., Separation of Concerns]
- [Rule about boundaries. E.g., "Domain logic must not import framework, database, or HTTP packages."]
- [Rule about direction. E.g., "Dependencies flow inward: infrastructure → application → domain. Never the reverse."]
- [Rule about naming. E.g., "Name interfaces after their role (UserRepository), not their implementation (PrismaUserRepository)."]

### [Application — e.g., Where This Practice Applies]
- [Rule about when to apply. E.g., "Apply to every bounded context — not just the 'important' ones. Inconsistency creates confusion."]
- [Rule about granularity. E.g., "One use case per service method — don't combine unrelated operations."]
- [Rule about testing. E.g., "Unit test use cases with mocked ports; integration test adapters against real infrastructure."]

### [Common Scenarios — e.g., Introducing New Features]
- [Rule about new feature workflow. E.g., "Define the port first (interface), then write the use case against it, then implement the adapter."]
- [Rule about refactoring. E.g., "Don't refactor architecture and behavior in the same PR — isolate structural changes."]
- [Rule about shared code. E.g., "Shared kernel code must be explicitly agreed on — don't silently promote internal types to shared."]

### [Boundaries and Edge Cases]
- [Rule about what breaks the practice. E.g., "If you find yourself importing a framework type in a domain entity, stop — you've crossed a boundary."]
- [Rule about pragmatic exceptions. E.g., "For scripts and CLIs, full hexagonal setup is overkill — apply the principle, not the ceremony."]

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Domain importing ORM entity directly | Domain coupled to persistence detail | Define a port interface; adapter maps ORM → domain |
| Use case depending on `Request` / HTTP context | Use case tied to transport layer | Pass explicit parameters, not request objects |
| Shared mutable global state | Non-deterministic behavior, hard to test | Pass state explicitly or inject via constructor |
| Fat service with 10+ methods | Single Responsibility violated | Split by operation or bounded context |
| Testing implementation instead of behavior | Tests break on refactor, not on bugs | Assert outputs and side effects, not internal calls |
| Inconsistent application of pattern | Cognitive overhead, no clear mental model | Apply uniformly or not at all |

---

## Review Criteria

- [ ] Domain layer has no imports from infrastructure, framework, or transport packages
- [ ] Use cases accept explicit inputs — no request objects, no HTTP context
- [ ] Each use case has a single, clear responsibility
- [ ] Ports (interfaces) are defined in domain; adapters implement them in infrastructure
- [ ] Tests for use cases use mocked ports — not real database/HTTP
- [ ] Naming reveals intent — ports named by role, not technology
- [ ] No shared mutable state between use cases or modules

---

## Trade-offs

**[This practice] vs simplicity for small projects**: [Recommendation. E.g., "For a script or internal tool with one developer, full hexagonal setup adds ceremony without benefit. Apply the principles (separation, interfaces) without the full directory ceremony."]

**Strict vs pragmatic application**: [Recommendation. E.g., "Strict boundaries are mandatory at I/O (DB, HTTP, queue). Pure domain transformations can be simpler functions — don't over-engineer."]

**Monorepo vs separate packages for domain**: [Recommendation. E.g., "A monorepo with explicit package boundaries is a good middle ground — shared tooling, enforced separation."]

---

## Implementation Notes

- [Directory structure convention for this practice. E.g., "`src/domain/`, `src/application/`, `src/infrastructure/`, `src/presentation/`"]
- [Naming convention. E.g., "Use case files: `create-user.use-case.ts`. Port files: `user.repository.ts`. Adapter files: `prisma-user.repository.ts`."]
- [Linting or tooling to enforce boundaries. E.g., "Use `eslint-plugin-boundaries` to enforce import rules between layers."]
- [Common onboarding confusion. E.g., "New developers often put use cases in controllers or adapters in domain — add a short ADR explaining the rule."]
- [When to deviate. E.g., "CLI scripts, data migration scripts, and test fixtures don't need full layering — but should still avoid importing framework in domain."]
