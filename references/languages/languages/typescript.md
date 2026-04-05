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
- Use `satisfies` operator to validate shapes without widening (TS 4.9+):
  ```ts
  const config = { env: 'production', port: 3000 } satisfies Config;
  ```
- Use `const` type parameters (TS 5.0+) to preserve literal types in generic functions:
  ```ts
  function identity<const T>(value: T): T { return value; }
  const x = identity(['a', 'b']); // type: readonly ["a", "b"]
  ```
- Use `NoInfer<T>` (TS 5.4+) to prevent unintended generic inference from specific arguments:
  ```ts
  function setDefault<T>(items: T[], def: NoInfer<T>): T[] { ... }
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

### Explicit Resource Management (TS 5.2+)
- Use `using` for synchronous disposable resources and `await using` for async ones:
  ```ts
  using db = openConnection(); // calls db[Symbol.dispose]() on scope exit
  await using stream = openStream(); // calls stream[Symbol.asyncDispose]()
  ```
- Implement `[Symbol.dispose]()` on any resource class that manages a connection, handle, or subscription.

### Modules
- Use barrel files (`index.ts`) only at public API boundaries — avoid for internal modules.
- Import paths must be explicit. Avoid wildcard re-exports that pollute consumers.
- Use `--verbatimModuleSyntax` (TS 5.0+) to ensure `import type` is explicit — prevents runtime import of type-only dependencies.

### Generics
- Name generic parameters beyond `T`, `U` only when they represent domain concepts: `TEntity`, `TKey`.
- Constrain generics as tightly as practical: `<T extends Record<string, unknown>>` over `<T>`.

### Decorators (TS 5.0+)
- Use the new TC39 Stage 3 decorators syntax (TS 5.0), not the legacy `experimentalDecorators`.
- Decorators are for cross-cutting concerns only (logging, validation, DI) — not business logic.
- Do not mix legacy `experimentalDecorators` and new decorators in the same project.

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
| `enum` for string constants | Generates runtime code, poor tree-shaking | Use string literal unions |
| `const` enum across module boundaries | Breaks with `isolatedModules` | Use regular union types or object `as const` |
| Skipping `using` for disposable resources | Resource leaks | Implement `Symbol.dispose` and use `using` |

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
- [ ] `satisfies` used instead of type assertion for literal validation
- [ ] `using`/`await using` applied to all disposable resources

---

## Trade-offs

**Interface vs Type**: `interface` merges (useful for extending third-party types), `type` does not. Prefer `type` by default in application code; use `interface` for extensible contracts.

**Enums vs Union Types**: TypeScript enums generate runtime code. String literal unions (`'pending' | 'active' | 'closed'`) are zero-cost and easier to refine. Prefer union types unless interop with external systems requires enum values.

**Zod vs Manual Validation**: Zod adds ~15KB but gives runtime safety + type inference at boundaries. Manual validation is lighter but error-prone. Use Zod at all external input boundaries (HTTP, env vars, storage).

**Legacy decorators vs TC39 decorators**: TC39 decorators (TS 5.0+) are the future standard. Migrate away from `experimentalDecorators` in new code. NestJS still requires legacy decorators — check framework compatibility before switching.

---

## Implementation Notes

- Always configure `tsconfig.json` with `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`.
- Add `"verbatimModuleSyntax": true` in TS 5.0+ projects to enforce explicit `import type` usage.
- Use `ts-reset` or `@total-typescript/ts-reset` to fix stdlib type holes.
- Keep `tsconfig.paths` minimal — deep path aliasing creates confusion in large codebases.
- Run `tsc --noEmit` in CI as a type-only check step separate from the build.
- TypeScript 5 requires Node.js 14.17+ — verify engine constraints in `package.json`.
