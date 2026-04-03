# [UtilityName] — Engineering Reference

> Utilities are cross-cutting concerns applied throughout the codebase — error handling, logging, testing, caching, etc.

## Best Practices

### [Core Behavior — e.g., When to Apply]
- [Rule about where this utility belongs in the architecture. E.g., "Apply at infrastructure adapters — never in domain or use-case layers."]
- [Rule about consistency. E.g., "Use the same logger interface everywhere — no direct imports of the logger package from domain code."]
- [Rule about configuration. E.g., "Configure via environment variables — never hardcode log level or error format."]

### [Pattern — e.g., Structuring Output / Format]
- [Rule about format. E.g., "Always log structured JSON in production — human-readable format is acceptable in development only."]
- [Rule about what to include. E.g., "Every log entry must include: timestamp, level, message, trace ID, and service name."]
- [Rule about what NOT to include. E.g., "Never log PII, secrets, or raw request bodies."]

### [Scope — e.g., Error Categories / Test Scope]
- [Rule about scope. E.g., "Distinguish expected errors (validation, not found) from unexpected errors (panics, DB failures) — handle differently."]
- [Rule about severity or priority. E.g., "Use WARN for recoverable issues; ERROR for failures that affect the user; FATAL only if the process must exit."]
- [Rule about test scope. E.g., "Unit tests cover pure logic; integration tests cover I/O boundaries; e2e tests cover critical user flows only."]

### [Edge Cases]
- [Rule about error propagation or failure modes]
- [Rule about retry, circuit breaker, or fallback strategy if applicable]
- [Rule about observability — what to measure or trace]

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Swallowing errors without logging | Silent failures, impossible to debug | Log before discarding, or return the error |
| `console.log` / `fmt.Println` in production code | Unstructured, unsearchable output | Use structured logger with levels |
| Testing implementation details | Tests break on refactor, not on bugs | Test behavior and outputs, not internal calls |
| No correlation / trace ID in logs | Cannot follow a request across services | Inject trace ID at entry point, propagate via context |
| Logging in domain layer | Domain coupled to infrastructure | Use interface injection or event-based logging |
| Catching all errors with the same handler | Loses specificity, wrong recovery path | Differentiate by error type or category |

---

## Review Criteria

- [ ] No direct infrastructure imports in domain or use-case layers for this utility
- [ ] Structured output format enforced (no raw string logging in production)
- [ ] No secrets or PII in logged/emitted output
- [ ] Trace/correlation ID present in all cross-service or cross-request operations
- [ ] Tests assert behavior, not internal implementation
- [ ] Error types are categorized — expected vs unexpected handled differently
- [ ] Utility is configured via environment, not hardcoded

---

## Trade-offs

**Centralized vs distributed [utility]**: [Recommendation. E.g., "Centralize the logger configuration and inject the instance — don't allow each module to configure its own logger."]

**Verbose logging vs signal-to-noise**: [Recommendation. E.g., "Log at DEBUG only in development. In production, log only what is needed to diagnose a failure — excessive logging degrades performance and raises storage costs."]

**Mocking vs real implementation in tests**: [Recommendation. E.g., "Mock external I/O (HTTP, DB); use real implementations for pure utility logic (string formatting, parsing, math)."]

---

## Implementation Notes

- [Package or library to use. E.g., "`zap` for Go logging — faster than `logrus`, structured by default."]
- [Setup or init step. E.g., "Initialize logger once at main/bootstrap and inject via DI — never call `zap.NewNop()` in production."]
- [Integration with observability stack. E.g., "Forward logs to Datadog/Loki via `OTEL_EXPORTER_OTLP_ENDPOINT`."]
- [Common configuration flags or env vars]
- [Gotcha or known issue. E.g., "Zap's `SugaredLogger` is convenient but has a performance cost in high-throughput paths — prefer `Logger` for hot code."]
