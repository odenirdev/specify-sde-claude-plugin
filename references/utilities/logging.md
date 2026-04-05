# Logging — Engineering Reference

## Core Principle

Logs exist for one purpose: enabling a human or system to understand what happened when something goes wrong in production. Treat logs as an operational tool, not a debugging trace.

## Best Practices

### Structured Logging

Always log structured JSON, never plain strings:
```ts
// Bad
logger.info(`User ${userId} created successfully`);

// Good
logger.info({ event: 'user.created', userId, email }, 'User created');
```

Structured logs are queryable, filterable, and indexable by log aggregators (Datadog, Loki, CloudWatch).

### Log Levels

| Level | When to use |
|---|---|
| `error` | Unexpected failures that require attention |
| `warn` | Degraded behavior, fallbacks, retries exceeded |
| `info` | Significant state transitions (started, completed, failed) |
| `debug` | Detailed trace for debugging — disabled in production |

Never use `console.log` in production code. Never leave debug logs in shipped code.

### What to Log

Always include:
- `event` — machine-readable identifier (e.g., `payment.charged`, `auth.token_refreshed`)
- Operation identifiers: `userId`, `orderId`, `requestId`
- Result: `success`, `failure`, `skipped`
- Duration for operations with latency expectations

Never log:
- Passwords, tokens, secrets, PII (email, CPF, credit card)
- Full request/response bodies by default (use sampling if needed)
- Stack traces at `info` level (only at `error` level)

### Request Correlation

Propagate a `requestId` or `traceId` through all log entries for a request lifecycle. Use middleware to attach it to the logger context:
```ts
app.use((req, res, next) => {
  req.log = logger.child({ requestId: req.headers['x-request-id'] ?? ulid() });
  next();
});
```

### Critical Operation Logging

Log start + end for long-running or critical operations:
```ts
logger.info({ event: 'sync.started', source, targetCount }, 'Sync started');
try {
  await runSync();
  logger.info({ event: 'sync.completed', source, duration }, 'Sync completed');
} catch (err) {
  logger.error({ event: 'sync.failed', source, err }, 'Sync failed');
  throw err;
}
```

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| `console.log('here')` | Not structured, not leveled, pollutes output |
| Logging PII | Compliance violation, security risk |
| Log on every function call | Noise makes real signals invisible |
| `logger.info(err.stack)` | Stack traces belong at `error` level |
| String interpolation in log messages | Not queryable by structured fields |

---

## Review Criteria

- [ ] No `console.log` in production code
- [ ] Structured fields include `event`, relevant IDs
- [ ] No sensitive data in log payloads
- [ ] `error` logs include the error object (not just message)
- [ ] Request/trace ID propagated through lifecycle
- [ ] Critical operations log start and completion/failure

---

## Implementation Notes

- Use `pino` (Node.js) or `zap` (Go) for structured, high-performance logging.
- Configure log level via environment variable: `LOG_LEVEL=debug` for development.
- Use `pino-pretty` in development for human-readable output; structured JSON in production.
- Never log at `debug` level in production by default — enable on-demand via config.
