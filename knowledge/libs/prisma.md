# Prisma — Engineering Reference

## Best Practices

### Schema Design
- Always define explicit `@@map` and `@map` to control table and column names independently from model names.
- Use `@id @default(cuid())` or `@default(uuid())` for IDs — never auto-increment integers for distributed systems.
- Mark fields as `@updatedAt` where applicable — don't manage timestamps manually.
- Prefer explicit nullable (`?`) over absent fields. Ambiguity in nullability creates subtle bugs.
- Use `@@index` for fields queried in WHERE or ORDER BY clauses.

### Client Usage
- Never instantiate `new PrismaClient()` in request handlers — create a singleton.
- In tests, use a dedicated test database or transaction rollback strategy.
- Always handle `PrismaClientKnownRequestError` explicitly:
  ```ts
  import { PrismaClientKnownRequestError } from '@prisma/client/runtime/library';

  try {
    return await prisma.user.create({ data });
  } catch (err) {
    if (err instanceof PrismaClientKnownRequestError && err.code === 'P2002') {
      throw new ConflictError('Email already registered');
    }
    throw err;
  }
  ```
- Use `prisma.$transaction` for operations that must be atomic.

### Queries
- Select only needed fields with `select` or `include` — never return full model when a subset suffices.
- Use cursor-based pagination for large datasets instead of `skip/take` with high offsets.
- Avoid N+1: use `include` or `findMany` with `where: { id: { in: ids } }` instead of looping queries.

### Migrations
- Never edit migration files after they've been applied to any environment.
- Use descriptive migration names: `npx prisma migrate dev --name add_refresh_token_to_user`.
- Review generated SQL before applying in production: `npx prisma migrate diff`.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `new PrismaClient()` per request | Connection pool exhaustion | Module-level singleton |
| Uncaught `PrismaClientKnownRequestError` | Unhandled constraint violations | Map known error codes to domain errors |
| `findMany` without pagination | OOM on large tables | Always paginate or use cursor |
| `include: { everything: true }` | Over-fetching | Select only what's needed |
| Raw `$queryRaw` without parameterization | SQL injection | Use tagged template literals: `` $queryRaw`...` `` |

---

## Review Criteria

- [ ] PrismaClient is a singleton
- [ ] Constraint errors (P2002, P2025) are mapped to domain errors
- [ ] No unbounded `findMany` without `take` or cursor
- [ ] Transactions used for multi-step writes
- [ ] `select` or `include` scoped to required fields only
- [ ] Migration files are not edited after creation

---

## Trade-offs

**Prisma vs Drizzle**: Prisma is more opinionated and generates a typed client from schema. Drizzle is SQL-first and gives more control over queries. Prisma wins for teams that want guardrails; Drizzle wins for teams that want SQL expressiveness.

**Schema-first vs Code-first**: Prisma is schema-first — the `schema.prisma` is the single source of truth. This is an asset: schema changes are explicit and reviewable.

---

## Implementation Notes

- Add `prisma/migrations` to version control.
- In CI, run `npx prisma migrate deploy` (not `dev`) for production deployments.
- Use `prisma generate` as a postinstall script to keep the client synchronized.
- For multi-tenant systems, consider row-level security or separate schemas — Prisma has limited native support for RLS.
