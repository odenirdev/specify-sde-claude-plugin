# Hexagonal Architecture — Engineering Reference

## Core Concept

Hexagonal architecture (Ports & Adapters) separates the application core from external systems. The core never knows what delivers requests to it or where it stores data. External systems are swappable by replacing adapters.

## Dependency Rule

```
Infrastructure → Application → Domain
```

- **Domain**: Pure business logic. No framework, no I/O, no HTTP, no database.
- **Application (Use Cases)**: Orchestrates domain operations. Depends on domain + ports (interfaces). No infrastructure.
- **Infrastructure (Adapters)**: Implements ports. Contains database clients, HTTP handlers, queue consumers, etc.

**The rule**: inner layers never import outer layers.

## Structure

```
src/
  domain/
    user/
      user.entity.ts         # Domain model (pure TypeScript class)
      user.repository.ts     # Port (interface) — defines contract
      user.service.ts        # Domain service (business invariants)

  application/
    use-cases/
      create-user.use-case.ts   # Depends on UserRepository interface
      find-user.use-case.ts

  infrastructure/
    persistence/
      prisma-user.repository.ts  # Implements UserRepository
    http/
      user.controller.ts         # Maps HTTP to use case calls
    config/
      di-container.ts            # Wires ports to adapters
```

## Ports

A port is an interface that the application defines:
```ts
// domain/user/user.repository.ts (port)
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
}
```

## Adapters

An adapter implements a port:
```ts
// infrastructure/persistence/prisma-user.repository.ts (adapter)
export class PrismaUserRepository implements UserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.prisma.user.findUnique({ where: { id } });
    return row ? UserMapper.toDomain(row) : null;
  }
  // ...
}
```

## Use Cases

Use cases depend on ports, never on adapters:
```ts
// application/use-cases/create-user.use-case.ts
export class CreateUserUseCase {
  constructor(
    private readonly users: UserRepository,  // interface, not concrete class
    private readonly hasher: PasswordHasher, // interface
  ) {}

  async execute(input: CreateUserInput): Promise<User> {
    const existing = await this.users.findByEmail(input.email);
    if (existing) throw new ConflictError('Email already registered');

    const user = User.create({ ...input, password: await this.hasher.hash(input.password) });
    await this.users.save(user);
    return user;
  }
}
```

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Use case imports `PrismaClient` directly | Domain depends on infrastructure |
| Controller contains business logic | Violates SRP; untestable without HTTP |
| Repository returns ORM entity to use case | Leaks infrastructure shape into domain |
| Domain model has `@Column()` decorators | Framework coupled to domain |
| Service imports `Request` from Express/NestJS | Application layer depends on delivery layer |

---

## Review Criteria

- [ ] Domain layer has zero framework imports
- [ ] Use cases depend on interfaces, not concrete classes
- [ ] Adapters implement domain-defined ports
- [ ] Controllers map HTTP → use case → HTTP (no logic)
- [ ] Domain entities are mapped to/from ORM models at the adapter boundary
- [ ] Tests inject fakes, not real adapters

---

## Trade-offs

**Hexagonal vs Layered**: Classic N-tier (controller → service → repository) is simpler but conflates use cases with infrastructure. Hexagonal is more code but the domain is portable and independently testable.

**When NOT to use**: Small scripts, CLIs, throwaway code. The overhead isn't worth it when there's no domain to protect.

---

## Implementation Notes

- Use dependency injection containers (NestJS DI, tsyringe, Wire for Go) to wire ports to adapters.
- In tests, inject fake implementations of ports — never mock concrete classes.
- Keep `UserMapper` close to the adapter — it handles the translation between infrastructure and domain shapes.
