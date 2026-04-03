# TypeScript — Engineering Reference

## Best Practices

### Type System
- Prefer `interface` for object shapes that can be extended; use `type` for unions, intersections, and aliases.
- Never use `any`. Use `unknown` when type is genuinely unknown and narrow it explicitly.
- Use `as const` for literal object/array constants to preserve literal types.
- Prefer discriminated unions over optional fields to model state:
  ```ts
  // Bad
  type Result = { data?: User; error?: string };

  // Good
  type Result = { ok: true; data: User } | { ok: false; error: string };
  ```
- Use `satisfies` operator to validate shapes without widening:
  ```ts
  const config = { env: 'production', port: 3000 } satisfies Config;
  ```

### Functions
- Return explicit types on exported functions — never rely on inference for public API.
- Use function overloads instead of union parameters when behavior differs:
  ```ts
  function find(id: string): User;
  function find(filter: UserFilter): User[];
  ```
- Avoid optional parameters when a required parameter with a union type communicates intent better.

### Async
- Always `await` promises. Never fire-and-forget unless intentional, and document it.
- Wrap top-level async calls in explicit error handlers.
- Use `Promise.all` for independent concurrent operations; never sequential `await` in loops.

### Modules
- Use barrel files (`index.ts`) only at public API boundaries — avoid for internal modules.
- Import paths must be explicit. Avoid wildcard re-exports that pollute consumers.

### Generics
- Name generic parameters beyond `T`, `U` only when they represent domain concepts: `TEntity`, `TKey`.
- Constrain generics as tightly as practical: `<T extends Record<string, unknown>>` over `<T>`.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `as SomeType` casting | Hides type errors | Narrow with guards or validate at boundary |
| `!` non-null assertion | Lies about runtime value | Return early or throw on null |
| `any` | Defeats type safety | Use `unknown` + narrowing |
| `// @ts-ignore` | Suppresses errors silently | Fix root cause or use `@ts-expect-error` with explanation |
| Nested callbacks | Async hell | `async/await` with proper error handling |
| Mutable exported objects | Shared state bugs | Export `readonly` or freeze |

---

## Review Criteria

When reviewing TypeScript code, check:
- [ ] No `any` without justification comment
- [ ] All public function signatures have explicit return types
- [ ] Discriminated unions used for state variants
- [ ] Error paths return typed errors, not `undefined`
- [ ] Generics have constraints
- [ ] Async code uses `await`, not `.then()` chaining without catch
- [ ] `strictNullChecks` compatible — no unsafe member access on potentially null values

---

## Trade-offs

**Interface vs Type**: `interface` merges (useful for extending third-party types), `type` does not. Prefer `type` by default in application code; use `interface` for extensible contracts.

**Enums vs Union Types**: TypeScript enums generate runtime code. String literal unions (`'pending' | 'active' | 'closed'`) are zero-cost and easier to refine. Prefer union types unless interop with external systems requires enum values.

**Zod vs Manual Validation**: Zod adds ~15KB but gives runtime safety + type inference at boundaries. Manual validation is lighter but error-prone. Use Zod at all external input boundaries (HTTP, env vars, storage).

---

## Implementation Notes

- Always configure `tsconfig.json` with `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`.
- Use `ts-reset` or `@total-typescript/ts-reset` to fix stdlib type holes.
- Keep `tsconfig.paths` minimal — deep path aliasing creates confusion in large codebases.
