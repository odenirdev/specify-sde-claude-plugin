# NestJS — Engineering Reference

## Best Practices

### Module Structure
- One feature = one module. Don't create a single `AppModule` that imports everything.
- Each module owns its domain: controller, service, repository, DTOs.
- Export only what other modules need to consume — keep the surface small.
- Use `forRoot`/`forRootAsync` for global infrastructure (database, config, cache).

### Controllers
- Controllers are thin routing layers — no business logic.
- Validate all incoming data with DTOs + `class-validator`. Never trust `req.body` directly.
- Use `@ApiTags`, `@ApiOperation`, `@ApiResponse` for Swagger documentation.
- Return domain objects, not ORM entities — map before responding.

### Services (Use Cases)
- Services contain application logic. One service per use case is clean but pragmatic — group related operations.
- Services must not depend on `Request`, HTTP context, or controllers.
- Inject interfaces (ports), not concrete implementations. Wire in modules.
- Keep services testable: all I/O through injected abstractions.

### DTOs and Validation
```ts
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```
- Use `ValidationPipe` globally with `whitelist: true, forbidNonWhitelisted: true`.
- Never share input DTOs as response types — define separate response classes or use `@Exclude`.

### Exception Handling
- Throw NestJS `HttpException` subclasses in controllers/guards.
- Services throw domain errors (custom `Error` subclasses), never HTTP errors.
- Use a global `ExceptionFilter` to map domain errors to HTTP responses.

### Dependency Injection
- Use `CUSTOM_TOKEN` injection tokens for interface bindings:
  ```ts
  { provide: USER_REPOSITORY, useClass: PrismaUserRepository }
  ```
- Avoid `@Inject(SomeConcreteClass)` in services — that's a hidden coupling.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Business logic in controllers | Untestable, coupled to HTTP | Move to service |
| ORM entity returned from endpoint | Leaks internals, breaks on schema change | Map to response DTO |
| `any` typed request body | No validation | Use validated DTO |
| Importing `HttpException` in service | Domain depends on HTTP | Service throws domain error; filter maps it |
| Circular module imports | DI resolution fails | Introduce shared module or restructure |

---

## Review Criteria

- [ ] Controllers contain no business logic
- [ ] All DTOs use `class-validator` decorators
- [ ] `ValidationPipe` is enabled globally with whitelist
- [ ] Services inject interfaces, not concrete classes
- [ ] Domain errors are separated from HTTP errors
- [ ] ORM entities are not exposed directly via endpoints
- [ ] Modules are feature-scoped, not god modules

---

## Trade-offs

**TypeORM vs Prisma in NestJS**: TypeORM integrates natively with decorators. Prisma is more explicit, type-safe, and migration-friendly. For greenfield projects, Prisma is the better default.

**REST vs GraphQL**: GraphQL adds flexibility for complex queries but increases surface area, resolver complexity, and N+1 risk. Prefer REST for internal APIs and GraphQL only when client flexibility is explicitly needed.

**Microservices vs Monolith**: NestJS supports both. Start as a modular monolith. Extract services only when deployment, scaling, or team boundaries justify it.

---

## Implementation Notes

- Always run in strict TypeScript mode.
- Use `ConfigModule` with `Joi` or `Zod` to validate env vars at startup.
- Use `@nestjs/swagger` to keep API docs synchronized with implementation.
- Lifecycle hooks (`onModuleInit`, `onApplicationBootstrap`) for startup checks, not constructor side effects.
- Health endpoints via `@nestjs/terminus` for load balancer readiness/liveness checks.
