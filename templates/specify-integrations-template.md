# Integrations — [Project Name]

> External services and their contracts. Updated when integrations are added, changed, or removed.

---

## [Service Name]

**Purpose**: [What this service does for us — one sentence]
**Type**: [REST API / Webhook / SDK / Queue / etc.]
**Environment**: [Production URL pattern or region — no secrets here]

### Authentication
- **Method**: [API Key / OAuth2 / mTLS / HMAC signature]
- **Where configured**: [Environment variable name(s)]

### Key Operations

| Operation | Endpoint / Method | Notes |
|---|---|---|
| [e.g., Create charge] | `POST /v1/charges` | Idempotency key required |
| [e.g., Retrieve customer] | `GET /v1/customers/{id}` | |

### Timeout & Retry Policy
- **Timeout**: [e.g., 10s]
- **Retries**: [e.g., 3 attempts with exponential backoff on 5xx and network errors]
- **Circuit breaker**: [If applicable]

### Failure Behavior
- **[Error type]**: [How the system handles it — e.g., "422: throws PaymentValidationError, no retry"]
- **[Error type]**: [e.g., "503: retried 3x, then throws ServiceUnavailableError"]
- **[Timeout]**: [e.g., "Treated as transient, retried once, then alert"]

### Incoming Webhooks (if applicable)
- **Endpoint**: `POST /webhooks/[service]`
- **Signature verification**: [How the signature is verified]
- **Idempotency**: [How duplicate events are handled]

### Notes
[Any non-obvious integration behavior, known limitations, or operational observations]

---

## [Service Name 2]

...

---

## Decommissioned Integrations

| Service | Removed | Reason |
|---|---|---|
| [Service] | YYYY-MM-DD | [Migration to X / deprecated / unused] |
