# Architecture — [Project Name]

> High-level system design. Updated when significant structural changes are made. Not a complete implementation guide — a navigation map.

---

## System Overview

[One paragraph describing the system's main architectural characteristics and the key design decisions that shaped it.]

---

## Component Map

```
[ASCII diagram or structured description of main components and their relationships]

Example:
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│   HTTP Layer    │────▶│  Application     │────▶│  Domain     │
│ (Controllers)   │     │  (Use Cases)     │     │  (Entities) │
└─────────────────┘     └──────────────────┘     └─────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐     ┌──────────────────┐
│   Auth / Guard  │     │  Infra Adapters  │
│                 │     │  (DB, Queue, HTTP)│
└─────────────────┘     └──────────────────┘
```

---

## Components

### [Component Name]
- **Responsibility**: [What this component owns]
- **Interfaces**: [What it exposes to other components]
- **Dependencies**: [What it depends on]
- **Location**: [Directory path in the codebase]

### [Component Name]
- **Responsibility**: ...
- ...

---

## Data Flow

### [Main Flow Name — e.g., "Process Order"]
1. HTTP request hits `OrderController`
2. Controller validates DTO and calls `CreateOrderUseCase`
3. Use case checks inventory via `InventoryRepository` interface
4. Use case creates `Order` domain entity and persists via `OrderRepository`
5. Domain event published to queue
6. Response mapped to DTO and returned

### [Secondary Flow Name]
...

---

## Key Boundaries

| Boundary | What crosses it | Contract |
|---|---|---|
| HTTP → Application | Validated DTOs | REST API (see integrations.md) |
| Application → Infrastructure | Repository interfaces | Domain interfaces |
| Service → Queue | Domain events | [Schema/format] |

---

## Consistency Model

- **Transactional boundary**: [e.g., "per HTTP request, within a single service"]
- **Eventual consistency**: [What parts of the system are eventually consistent and why]
- **Idempotency**: [How duplicate operations are handled]

---

## Architectural Decisions

| ADR | Decision | Status |
|---|---|---|
| [ADR-001](./decisions/ADR-001-[name].md) | [Summary] | Accepted |

---

## Known Constraints

- [Constraint 1 — e.g., "Must support 10K concurrent connections"]
- [Constraint 2 — e.g., "Cannot modify shared schema — legacy dependency"]
