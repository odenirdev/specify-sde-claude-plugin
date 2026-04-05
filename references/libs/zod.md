# Zod — Engineering Reference

## Best Practices

### Schema Definition
- Define schemas at system boundaries where data is untrusted: HTTP requests, environment config, external APIs, persisted JSON, and form submissions.
- Export both the schema and the inferred TypeScript type from the same module:
  ```ts
  import { z } from 'zod';

  export const LoginSchema = z.object({
    email: z.string().email(),
    password: z.string().min(8),
  });

  export type LoginInput = z.infer<typeof LoginSchema>;
  ```
- Keep schemas at module scope, never recreated inside React renders or hot code paths.
- Enable TypeScript `strict` mode — Zod works best in strict projects.

### Parsing Strategy
- Use `.parse()` when invalid input should fail fast and be treated as an explicit error.
- Use `.safeParse()` when the caller needs a success/error result object without throwing.
- Format validation errors before returning them to the UI or logs; do not leak raw internal details to end users.
- Use `.refine()` or `.superRefine()` for domain invariants that cannot be expressed by primitive rules alone.

### Reuse & Composition
- Compose schemas from smaller building blocks instead of duplicating shapes.
- Reuse shared schemas across frontend and backend when the contract is truly shared.
- Prefer explicit transformations and coercions only when the conversion is intentional and well understood.

### Error Handling
- Treat schema validation failures as normal boundary errors, not unexpected crashes.
- Log enough context for debugging, but keep user-facing messages concise and safe.
- Convert external data to trusted app types immediately after parsing; avoid passing raw unknown payloads deeper into the domain.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Trusting external input because TypeScript compiles | TS types do not validate runtime data | Parse data with a Zod schema at the boundary |
| Defining schemas inline inside components/functions | Recreated objects, inconsistent reuse | Move schemas to module scope |
| Overusing `optional().nullable()` everywhere | Contract becomes vague and hard to reason about | Model nullability explicitly and narrowly |
| Blind coercion of user input | Unexpected values silently pass through | Coerce only when product rules require it |
| Casting with `as SomeType` after failed/no validation | Hides invalid runtime data | Use `parse` / `safeParse` and handle the result |

---

## Review Criteria

- [ ] All untrusted external input is validated at the boundary
- [ ] Schemas live in reusable modules, not inline renders
- [ ] `z.infer` is used instead of duplicated manual interfaces when appropriate
- [ ] `.parse()` vs `.safeParse()` choice is intentional
- [ ] Validation errors are handled explicitly and safely
- [ ] Nullability/optionality is modeled precisely, not loosely

---

## Trade-offs

**Zod vs TypeScript interfaces alone**: interfaces help at compile time only; Zod adds runtime guarantees for external data.

**`.parse()` vs `.safeParse()`**: `.parse()` is concise and good for fail-fast boundaries; `.safeParse()` is better when the caller needs to accumulate or display validation feedback.

---

## Implementation Notes

- Zod 4 is current and stable, but many codebases still use Zod 3 APIs. Check the installed version before adopting newer features.
- Zod pairs well with frontend form validation and backend request/env validation, enabling one source of truth for schemas.
- Keep schemas intention-revealing and close to the domain/use-case boundary they protect.
