# Error Handling — Engineering Reference

## Core Principle

Every error must travel up the call stack with enough context to be understood without reading the source code. Silent swallowing, empty catch blocks, and fallback default returns are defects — not defensive coding.

## Patterns

### Contextual Error Wrapping

Always add context when re-throwing:
```ts
// TypeScript
try {
  return await userRepo.findById(id);
} catch (err) {
  throw new Error(`FindUser id=${id}: ${(err as Error).message}`, { cause: err });
}
```
```go
// Go
user, err := repo.FindByID(ctx, id)
if err != nil {
    return nil, fmt.Errorf("FindUser id=%s: %w", id, err)
}
```

### Domain Error Types

Define typed errors for known failure cases:
```ts
export class NotFoundError extends Error {
  constructor(resource: string, id: string) {
    super(`${resource} not found: ${id}`);
    this.name = 'NotFoundError';
  }
}

export class ConflictError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ConflictError';
  }
}
```
Map these to HTTP at the adapter layer — never in domain logic.

### Boundary Mapping

At every system boundary (HTTP, queue, external service), map infrastructure errors to domain errors:
```ts
// HTTP adapter
export function toHttpError(err: unknown): HttpException {
  if (err instanceof NotFoundError) return new NotFoundException(err.message);
  if (err instanceof ConflictError) return new ConflictException(err.message);
  // Don't expose internal details
  return new InternalServerErrorException('Unexpected error');
}
```

### Result Type (optional, explicit)

For operations where failure is expected and not exceptional:
```ts
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

function parseDate(raw: string): Result<Date, string> {
  const d = new Date(raw);
  if (isNaN(d.getTime())) return { ok: false, error: `Invalid date: ${raw}` };
  return { ok: true, value: d };
}
```
Use Result types at domain logic boundaries, not as a replacement for exceptions on exceptional cases.

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| `catch (err) {}` empty catch | Error disappears, problem is invisible |
| `catch (err) { return null }` | Caller has no way to distinguish success from failure |
| `console.error` then continue | Error is logged but program state is corrupt |
| Generic `throw new Error('something went wrong')` | Impossible to programmatically respond |
| Catching `Error` when you need `AxiosError` | Wrong type guard, causes crashes |

---

## Review Criteria

- [ ] All catch blocks either re-throw with context OR map to a typed domain error
- [ ] No empty catch blocks
- [ ] No silent `return null` / `return undefined` on error
- [ ] Domain errors defined for all known failure modes
- [ ] Infrastructure errors mapped to domain errors at boundary
- [ ] External error messages not exposed to end users

---

## Trade-offs

**Exceptions vs Result types**: Exceptions are ergonomic for truly exceptional conditions (infra failures, unexpected state). Result types are explicit for expected failures (validation, not found). Mix both based on semantics — not religiously.

**Error logging at throw vs catch**: Log at the boundary where you handle the error for HTTP response, not at every re-throw. Otherwise you get duplicate log entries.
