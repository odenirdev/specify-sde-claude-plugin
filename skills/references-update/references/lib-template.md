# [LibName] — Engineering Reference

## Best Practices

### [Core Concept 1 — e.g., Schema / Model Definition]
- [Rule about where to define schemas/models. E.g., "Define schema in one place, generate types from it — never hand-write types that mirror the schema."]
- [Rule about naming conventions for the library's artifacts]
- [Rule about co-location vs central definition]

### [Core Concept 2 — e.g., Querying / Data Access]
- [Rule about query complexity. E.g., "Prefer select over findMany without include — never fetch more than needed."]
- [Rule about N+1 prevention. E.g., "Use include/join at query level, not in a loop."]
- [Rule about pagination. E.g., "Always paginate list queries — never return unbounded results."]

### [Core Concept 3 — e.g., Error Handling]
- [Rule about catching library-specific errors. E.g., "Catch `PrismaClientKnownRequestError` at the adapter layer, not in use cases."]
- [Rule about mapping errors to domain errors]
- [Rule about not leaking library errors to callers]

### [Core Concept 4 — e.g., Connection / Client Management]
- [Rule about singleton client. E.g., "Instantiate one client per process — never create a new instance per request."]
- [Rule about connection pooling or lifecycle hooks]
- [Rule about disconnect/cleanup on shutdown]

### [Core Concept 5 — e.g., Migration / Schema Evolution]
- [Rule about migration workflow. E.g., "Run `migrate dev` in development, `migrate deploy` in CI — never `db push` in production."]
- [Rule about backward compatibility]
- [Rule about seeding or test data setup]

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Creating a new client instance per request | Connection pool exhaustion | Use singleton client via DI |
| Fetching full object when only ID is needed | Over-fetching, performance cost | Use `select: { id: true }` or equivalent |
| Catching library errors in domain/use-case layer | Domain coupled to infrastructure | Catch at adapter boundary, rethrow as domain error |
| Exposing raw library model as API response | Couples API shape to DB schema | Map to response DTO before returning |
| Running migrations manually in production | Inconsistent state, human error | Automate via CI pipeline |
| Using `any` as type for library results | Type safety lost | Use generated or inferred types from library |

---

## Review Criteria

- [ ] Single client instance shared across the application (no per-request instantiation)
- [ ] All queries have `select` or `include` scoped to what is needed — no unbounded includes
- [ ] Library-specific errors are caught only at the adapter/infrastructure layer
- [ ] Domain layer has no import from the library package
- [ ] List queries are paginated — no unbounded result sets
- [ ] Migrations run via CI, not manually
- [ ] Library models are mapped to DTOs before leaving the infrastructure layer

---

## Trade-offs

**[LibName] vs [Alternative]**: [Clear recommendation. E.g., "Prefer Prisma over TypeORM for new projects — better type inference, explicit migrations, and cleaner query API. Use TypeORM only when existing codebase already depends on it."]

**Code-first vs Schema-first**: [Recommendation. E.g., "Schema-first (Prisma schema, OpenAPI spec) produces a single source of truth — prefer it over code-first decorators that scatter schema definition across files."]

**Generated types vs manual types**: [Recommendation. Always use generated types — manual types diverge from reality and create false confidence.]

---

## Implementation Notes

- [Installation command. E.g., "`pnpm add @prisma/client && pnpm add -D prisma`"]
- [Init or setup command. E.g., "`npx prisma init --datasource-provider postgresql`"]
- [Generate step. E.g., "Add `prisma generate` to `postinstall` script so types are always up-to-date after install."]
- [Environment variable required. E.g., "`DATABASE_URL` must be set before any Prisma command runs."]
- [Key CLI command used daily. E.g., "`npx prisma studio` for quick data inspection in development."]
