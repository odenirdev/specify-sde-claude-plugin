# [FrameworkName] — Engineering Reference

## Best Practices

### [Core Concept 1 — e.g., Module / Route / Component Structure]
- [Specific, actionable rule. E.g., "One feature = one module. Don't create a god module."]
- [Rule about ownership or encapsulation]
- [Rule about what to export/expose]

### [Core Concept 2 — e.g., Request Handling / Data Flow]
- [Rule about thin layers — no business logic in controllers/handlers]
- [Rule about validation — never trust raw input]
- [Rule about response format — don't expose internals]

### [Core Concept 3 — e.g., Services / Use Cases]
- [Rule about single responsibility]
- [Rule about dependency injection or interface segregation]
- [Rule about testability — all I/O through abstractions]

### [Core Concept 4 — e.g., Error Handling]
- [Rule about where to throw vs where to catch]
- [Rule about framework errors vs domain errors]
- [Rule about global error handling setup]

### [Core Concept 5 — e.g., Configuration / Startup]
- [Rule about config validation at startup]
- [Rule about environment variables]
- [Rule about initialization order]

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Business logic in [controller/handler/route] | Untestable, coupled to transport layer | Move to service/use-case |
| [ORM entity / raw DB model] returned from endpoint | Leaks internals, breaks on schema change | Map to response DTO |
| No input validation | Runtime errors, security vulnerabilities | Validate at entry point with schema/decorator |
| [Framework error type] thrown in domain layer | Domain coupled to transport | Throw domain error; map at boundary |
| God [module/router/component] | Hard to navigate, breaks encapsulation | Split by feature |
| Direct dependency on concrete implementation | Untestable, brittle | Inject interface/port |

---

## Review Criteria

- [ ] [Controller/handler/route] contains no business logic
- [ ] All inputs are validated before reaching the service layer
- [ ] Domain layer has no imports from the framework package
- [ ] Dependencies are injected via interfaces, not concrete classes
- [ ] Domain errors are mapped to framework responses at the boundary only
- [ ] No ORM entities or internal models exposed via API response
- [ ] Config/env vars are validated at startup, not at runtime
- [ ] Modules/routes are feature-scoped, not tech-layer-scoped

---

## Trade-offs

**[FrameworkName] vs [Alternative]**: [Clear recommendation with rationale. E.g., "Prefer X for greenfield — better type safety and explicit migrations. Use Y only when team already knows it well."]

**REST vs GraphQL with [FrameworkName]**: [Recommendation. Default to REST unless client query flexibility is explicitly required — GraphQL adds resolver complexity and N+1 risk.]

**Monolith vs Microservices**: [Recommendation. Start modular monolith. Extract services only when deployment, scaling, or team boundaries justify it.]

---

## Implementation Notes

- [Specific CLI command or setup step. E.g., "Bootstrap with `npx @nestjs/cli new app --strict`"]
- [Required global config. E.g., "Enable `ValidationPipe` globally with `whitelist: true, forbidNonWhitelisted: true`"]
- [Key library or plugin. E.g., "Use `@nestjs/swagger` to keep API docs in sync with implementation"]
- [Startup check or lifecycle hook pattern]
- [Health check or observability setup]
