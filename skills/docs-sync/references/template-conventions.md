# Project Conventions — [Project Name]

> Conventions and patterns used in this codebase. Not rules for their own sake — documented because deviating from them creates inconsistency that costs maintainability.

---

## Project Structure

```
[Project structure tree — actual directory layout]

Example:
src/
  domain/           # Domain models, ports (interfaces), domain services
  application/      # Use cases
  infrastructure/   # Adapters: DB, HTTP clients, queue, etc.
  presentation/     # Controllers, DTOs, mappers
  config/           # DI wiring, environment config
```

---

## Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Files | kebab-case | `create-user.use-case.ts` |
| Classes | PascalCase | `CreateUserUseCase` |
| Interfaces/Ports | PascalCase, no `I` prefix | `UserRepository` |
| DTOs | PascalCase + suffix | `CreateUserDto`, `UserResponseDto` |
| Errors | PascalCase + `Error` | `ConflictError`, `NotFoundError` |
| DB tables | snake_case | `order_items` |
| Env vars | SCREAMING_SNAKE_CASE | `DATABASE_URL` |

---

## Architecture Conventions

- **Dependency flow**: `domain ← application ← infrastructure ← presentation`
- **Ports defined in**: `domain/[entity]/[entity].repository.ts`
- **Adapters implemented in**: `infrastructure/[concern]/[entity].repository.ts`
- **Use cases located in**: `application/use-cases/[verb]-[entity].use-case.ts`
- **Error mapping**: domain errors mapped to HTTP at controller or exception filter

---

## Error Handling Conventions

- Domain errors are plain `Error` subclasses defined in `domain/errors/`
- Infrastructure errors are caught at adapter boundaries and mapped to domain errors
- HTTP errors are produced at the presentation layer only
- No empty catch blocks — always handle or rethrow with context

---

## Testing Conventions

- Unit tests: `[file].spec.ts` in the same directory as source
- Integration tests: `[file].integration.spec.ts`
- Fakes: `[entity].repository.fake.ts` in `domain/[entity]/`
- Test data builders: `[entity].builder.ts`

---

## Git Conventions

- Branch: `[type]/[short-description]` — e.g., `feat/add-payment-integration`
- Commit: `type(scope): message` — e.g., `feat(payment): add Stripe charge endpoint`
- PR title follows commit format
- Every PR needs a spec link or description of what was changed and why

---

## Code Review Checklist

- [ ] Implementation matches spec (check `./.specify/specs`)
- [ ] No business logic in controllers
- [ ] Domain errors defined and mapped at boundary
- [ ] External calls have timeout and error handling
- [ ] Tests cover happy path + error cases
- [ ] No hardcoded config values

---

## Prohibited Patterns

- No `any` type without justification comment
- No `// @ts-ignore` without explanation
- No `console.log` in production code
- No hardcoded URLs, API keys, or config
- No direct database access in use cases (always through repository interface)
- No HTTP error classes in domain or application layers
