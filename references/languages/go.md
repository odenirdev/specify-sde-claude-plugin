# Go — Engineering Reference

## Best Practices

### Error Handling
- Never ignore errors. If you truly can't handle an error, propagate it upward with context:
  ```go
  user, err := repo.FindByID(ctx, id)
  if err != nil {
      return fmt.Errorf("FindUser id=%s: %w", id, err)
  }
  ```
- Use `errors.Is` and `errors.As` for error matching — never string comparison.
- Define sentinel errors for caller-controlled flow:
  ```go
  var ErrNotFound = errors.New("not found")
  ```
- Use `fmt.Errorf` with `%w` to wrap errors for stack-like context.

### Structs and Interfaces
- Define interfaces at the point of use (consumer), not at declaration (producer).
- Keep interfaces small — one or two methods is the Go ideal.
- Unexport fields that are not part of the public contract.

### Concurrency
- Never start goroutines inside library functions without an exit mechanism.
- Use `context.Context` as the first parameter in any blocking or cancellable operation.
- Synchronize with channels or `sync.Mutex`, never global variables.
- Prefer `errgroup` for managing goroutine fans with error collection.

### Functions
- Accept interfaces, return concrete types.
- Keep functions under 50 lines. Extract named helpers over inline complexity.
- Use table-driven tests instead of repetitive `TestFoo`, `TestFooEdge`, etc.

### Packages
- Package names are lowercase, single words. No `util`, `common`, `helpers`.
- Avoid circular imports — domain packages must not import infrastructure.
- Internal API: use `internal/` to prevent external consumption.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `_ = err` | Silent error discard | Handle or wrap + return |
| Global `var` mutable state | Race conditions, hard to test | Inject dependencies |
| Interface defined in producer package | Tight coupling | Define interface in consumer |
| `panic` for domain errors | Crashes service | Return `error` instead |
| `time.Sleep` in business logic | Flaky + untestable | Use ticker/timer + context |
| Empty `catch` equivalent (`if err != nil {}`) | Hides failures | Always handle or propagate |

---

## Review Criteria

- [ ] All errors wrapped with context via `fmt.Errorf("%w")`
- [ ] `context.Context` first arg in blocking calls
- [ ] No goroutine leak — all goroutines have a known exit path
- [ ] Interfaces defined at usage site
- [ ] No unused exports (exported identifiers are public API)
- [ ] Table-driven tests for cases beyond happy path
- [ ] No magic numbers — constants or enums used

---

## Trade-offs

**Mutex vs Channel**: Use mutexes to protect state, channels to communicate. "Do not communicate by sharing memory; share memory by communicating."

**Generics vs Interface**: Generics reduce allocations and avoid empty-interface conversions. Use when type safety and performance matter. Avoid over-generic code — it hurts readability.

**Sync vs Async IO**: Go's concurrency model means blocking IO is fine in goroutines. Don't add async abstraction layers unless coordinating multiple goroutines.

---

## Implementation Notes

- Run `gofmt` (or `goimports`) on every save. Non-negotiable.
- Use `golangci-lint` with `errcheck`, `govet`, `staticcheck` at minimum.
- Structure: `cmd/` for entrypoints, `internal/` for app logic, `pkg/` for reusable libraries.
- Use `//go:build` directives for platform-specific code; avoid `_linux.go` filename hacks.
- For configuration: use environment variables via `os.Getenv` or a typed config struct — never `init()` globals.
