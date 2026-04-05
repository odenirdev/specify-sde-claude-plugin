# API Design — Engineering Reference

## Core Principle

An API is a contract. Once published, changing it breaks consumers. Design for clarity, consistency, and evolvability from the start.

## REST Conventions

### Resource Naming
- Use nouns, not verbs: `/users`, not `/getUsers`
- Plural for collections: `/orders`, `/products`
- Nested resources for sub-entities: `/orders/{orderId}/items`
- Max two levels of nesting — beyond that, use query parameters
- kebab-case for multi-word paths: `/order-items`, not `/orderItems`

### HTTP Methods
| Method | Semantics | Idempotent |
|---|---|---|
| GET | Read resource(s) | Yes |
| POST | Create resource | No |
| PUT | Replace resource fully | Yes |
| PATCH | Partial update | No (by default) |
| DELETE | Remove resource | Yes |

Never use GET for mutations. Never use POST when PUT/PATCH/DELETE is semantically correct.

### Status Codes
- `200 OK` — successful read or update returning data
- `201 Created` — successful resource creation, include `Location` header
- `204 No Content` — successful action with no response body
- `400 Bad Request` — invalid input (validation error)
- `401 Unauthorized` — authentication required
- `403 Forbidden` — authenticated but lacks permission
- `404 Not Found` — resource not found
- `409 Conflict` — state conflict (duplicate, version mismatch)
- `422 Unprocessable Entity` — semantically invalid (valid syntax, business rule violation)
- `500 Internal Server Error` — unexpected server failure

### Error Response Shape
Standardize across all endpoints:
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Email is invalid",
  "details": [
    { "field": "email", "message": "Must be a valid email address" }
  ]
}
```
Never expose stack traces, internal error codes, or database errors to clients.

### Pagination
- Prefer cursor-based pagination for real-time data: `?after=cursor&limit=20`
- Use offset pagination only for stable datasets with moderate size
- Always include pagination metadata in response:
  ```json
  { "data": [...], "pagination": { "nextCursor": "abc123", "hasMore": true } }
  ```

### Versioning
- Version via URL path: `/v1/users` — explicit and cache-friendly
- Never break existing versions without deprecation period
- Use `Sunset` header to communicate deprecation timelines

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| `POST /getUser` | REST violation, confuses intent |
| 200 with error in body | Breaks HTTP semantics, hard to handle |
| Returning full entity on every response | Over-fetching, performance cost |
| Inconsistent naming (camelCase vs snake_case) | Confuses consumers |
| Unbounded list endpoints | OOM risk, poor performance |

---

## Review Criteria

- [ ] Resource names are nouns, plural
- [ ] HTTP method matches semantics
- [ ] Status codes are correct and consistent
- [ ] Error responses follow standard shape
- [ ] No stack traces or internal details in error responses
- [ ] List endpoints are paginated
- [ ] Auth and permission failures are distinguished (401 vs 403)

---

## Trade-offs

**REST vs GraphQL**: REST is simpler, cacheable, and better for public APIs. GraphQL is better for client-driven data fetching and reducing round trips. Never default to GraphQL — justify the added complexity.

**Versioning via URL vs Header**: URL versioning is explicit and easy to route. Header versioning is cleaner but harder to test and cache. Prefer URL versioning for external APIs.
