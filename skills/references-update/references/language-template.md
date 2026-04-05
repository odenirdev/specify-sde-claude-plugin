# [Language] — Engineering Reference

## Best Practices

### Type System
- [Rule about type strictness. E.g., "Always enable strict mode — no `any`, no implicit `undefined`."]
- [Rule about type inference vs explicit annotation. E.g., "Annotate function signatures explicitly; let the compiler infer local variable types."]
- [Rule about nullability. E.g., "Never use optional chaining as a substitute for proper null handling — if null is invalid, validate early."]

### Error Handling
- [Rule about error propagation. E.g., "Return errors explicitly — never swallow them in a catch block without logging or re-throwing."]
- [Rule about error types. E.g., "Use typed errors, not generic `Error` — define a domain error hierarchy."]
- [Rule about panic/exception vs return. E.g., "Reserve panic/exception for programmer errors; use return values for expected failure cases."]

### Module / Package Structure
- [Rule about module organization. E.g., "One package per bounded context — avoid utility packages that grow into god packages."]
- [Rule about circular dependencies. E.g., "Circular imports are a sign of wrong layer boundaries — restructure before continuing."]
- [Rule about exported surface. E.g., "Export only what external packages need — keep internals unexported."]

### Concurrency / Async
- [Rule about async model. E.g., "Prefer async/await over raw promise chains — it composes better and errors propagate correctly."]
- [Rule about shared state. E.g., "Never share mutable state across goroutines/threads without synchronization."]
- [Rule about cancellation. E.g., "Always propagate context/cancellation — never ignore it."]

### Testing
- [Rule about test isolation. E.g., "Tests must not depend on external state — mock or stub all I/O."]
- [Rule about naming. E.g., "Test names describe the scenario: `should return error when input is empty`."]
- [Rule about table-driven tests. E.g., "Use table-driven tests for multiple input/output pairs — avoid copy-paste test functions."]

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Using `any` / `interface{}` without justification | Type safety lost, runtime errors | Use specific types or generics |
| Swallowing errors (`_, err` or empty catch) | Silent failures, undebuggable | Log or return every error |
| Global mutable state | Race conditions, untestable | Pass state explicitly via arguments or DI |
| Blocking main thread / goroutine with I/O | Deadlocks, unresponsive service | Use async, goroutines, or worker threads correctly |
| Copying error strings instead of wrapping | Context lost in error chain | Wrap errors with context: `fmt.Errorf("...: %w", err)` |
| Circular package imports | Compilation failure or hidden coupling | Restructure — introduce shared abstraction |

---

## Review Criteria

- [ ] No `any` / `interface{}` without explicit justification in comment
- [ ] All errors are either returned or logged — no silent discards
- [ ] No global mutable state accessible across goroutines/threads
- [ ] Exported functions have explicit type signatures
- [ ] Tests are isolated — no shared state between test cases
- [ ] Cancellation context is propagated through all I/O calls
- [ ] Circular dependencies are absent

---

## Trade-offs

**Strict typing vs developer velocity**: [Recommendation. Strict typing always wins in the long run — early type errors are cheap; runtime errors in production are not.]

**Generics vs duplication**: [Recommendation. E.g., "Use generics when the same logic must work across types. Duplicate first, generalize when the pattern is stable — premature generics are hard to read."]

**Async/await vs callbacks vs raw promises**: [Recommendation. E.g., "Always prefer async/await — it composes, it's readable, and errors propagate through try/catch correctly."]

---

## Implementation Notes

- [Toolchain setup. E.g., "Use `gofmt` + `golangci-lint` in pre-commit — enforces formatting and catches common issues before CI."]
- [Required config file or flag. E.g., "`tsconfig.json` must have `strict: true` — no exceptions."]
- [Recommended IDE extension or plugin for the language]
- [Package manager or module command. E.g., "`go mod tidy` after every dependency change to keep `go.sum` clean."]
- [Common gotcha. E.g., "Zero values in Go are valid — always check whether zero means 'unset' or 'valid zero' in your domain."]
