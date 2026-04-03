# Axios — Engineering Reference

## Best Practices

### Instance Configuration
- Always create a named Axios instance per external service. Never use the default global instance.
- Configure `baseURL`, default `timeout`, and `headers` at creation time:
  ```ts
  const paymentClient = axios.create({
    baseURL: config.paymentServiceUrl,
    timeout: 10_000,
    headers: { 'Content-Type': 'application/json' },
  });
  ```
- Set a finite `timeout` on every instance. An infinite timeout will hang requests and exhaust connection pools.

### Error Handling
- Always use `axios.isAxiosError(err)` before accessing `.response`, `.request`, or `.config`:
  ```ts
  try {
    const { data } = await client.post('/charge', payload);
    return data;
  } catch (err) {
    if (axios.isAxiosError(err)) {
      const status = err.response?.status;
      if (status === 422) throw new ValidationError(err.response.data.message);
      if (status === 503) throw new ServiceUnavailableError('payment');
    }
    throw err;
  }
  ```
- Map HTTP status codes to domain errors at the HTTP client layer, not in callers.

### Interceptors
- Use request interceptors to attach auth tokens — don't pass tokens manually on every call.
- Use response interceptors to refresh expired tokens (retry once on 401).
- Log failed requests in response interceptors for observability:
  ```ts
  client.interceptors.response.use(
    (res) => res,
    (err) => {
      logger.error({ url: err.config?.url, status: err.response?.status }, 'HTTP request failed');
      return Promise.reject(err);
    }
  );
  ```
- Keep interceptors simple. Complex logic belongs in service layers.

### Cancellation
- Use `AbortController` for request cancellation (supported since Axios 0.22):
  ```ts
  const controller = new AbortController();
  const { data } = await client.get('/stream', { signal: controller.signal });
  // Later: controller.abort();
  ```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Default global `axios` instance | Shared config, side effects between modules | Dedicated instance per service |
| No timeout set | Requests hang forever | Set finite timeout on instance |
| Catching `err.response.data` without guard | Crashes if network error (no response) | `axios.isAxiosError` + `.response?.` |
| Swallowing non-2xx in interceptor | Silent failures | Reject with domain error |
| Hardcoded base URL | Not configurable | Inject from environment config |

---

## Review Criteria

- [ ] Named instance per external service
- [ ] Timeout configured on instance
- [ ] All catch blocks use `axios.isAxiosError` guard
- [ ] HTTP status codes mapped to domain errors
- [ ] Auth injected via interceptor, not manually
- [ ] No hardcoded URLs

---

## Trade-offs

**Axios vs fetch**: Native `fetch` is available everywhere but lacks interceptors, automatic JSON serialization, and typed errors. Axios adds ~20KB but provides a better DX for complex integrations. Use `fetch` for simple cases; Axios for services with auth, retries, or multiple interceptors.

**Instance per service vs shared**: Shared instances risk cross-service config bleed. One instance per external service is more explicit and testable.

---

## Implementation Notes

- Mock Axios in tests using `jest-mock-axios` or `msw` (Mock Service Worker). Never mock at the HTTP level in unit tests unless testing retry logic.
- For retry logic, use `axios-retry` with exponential backoff on 5xx and network errors.
- Type response data generically: `client.get<UserResponse>('/users')`.
