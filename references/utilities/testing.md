# Testing — Engineering Reference

## Core Principle

Tests are executable specifications. A passing test suite should give you confidence to deploy. A test that passes but doesn't detect real failures is worse than no test — it creates false confidence.

## Test Layers

### Unit Tests
- Test one function, one class, one use case in isolation.
- All I/O (database, HTTP, filesystem) is replaced by test doubles (fakes, stubs).
- Must be fast: < 5ms per test. No network. No disk.
- When to use: domain logic, utilities, transformations, validation.

### Integration Tests
- Test the interaction between two or more real components.
- Real database (test database, seeded), real file system, mocked external services.
- Acceptable latency: < 500ms per test.
- When to use: repository implementations, API endpoints, queue consumers.

### E2E Tests
- Test the full system as a user would.
- Real infrastructure, real external services (or stable stubs).
- Expensive — run in CI only, not on every commit.
- When to use: critical user flows, smoke tests, contract verification.

---

## Best Practices

### Arrange-Act-Assert (AAA)
```ts
it('returns NotFoundError when user does not exist', async () => {
  // Arrange
  const repo = new FakeUserRepository([]);
  const useCase = new FindUserUseCase(repo);

  // Act
  const result = useCase.execute('nonexistent-id');

  // Assert
  await expect(result).rejects.toThrow(NotFoundError);
});
```

### Test Naming
Name tests as specifications: `[subject] [condition] [expected outcome]`
```
✓ FindUser when user exists returns user data
✓ FindUser when id is empty throws ValidationError
✓ FindUser when user not found throws NotFoundError
```

### Fakes vs Mocks
- **Fakes**: Working in-memory implementations. Preferred for repositories and services.
- **Stubs**: Return fixed values for specific calls. Use for simple dependencies.
- **Mocks**: Verify interaction (calls, arguments). Use sparingly — they couple tests to implementation.
- Avoid `jest.spyOn` on implementation details. Test behavior, not internals.

### Determinism
- Never use `Date.now()`, `Math.random()`, or real network in tests.
- Inject clocks and random sources.
- Use fixed seeds for anything random.
- Tests must produce the same result on every run, in any environment.

### Coverage
- Coverage is a signal, not a goal. 90% coverage that misses the failure path is worthless.
- Prioritize: error paths, edge cases, boundary conditions.
- Every use case must have: 1 happy path, 1+ failure paths, 1+ edge cases.

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Testing implementation details | Tests break on refactor, not on bugs |
| Mocking what you own | Creates illusion of testing without real validation |
| `it('works')` with no descriptive name | Impossible to diagnose failures |
| `beforeEach` that's longer than the test | Test setup hides intent |
| Tests with no assertions | Pass always, test nothing |
| Sharing state between tests | Order-dependent failures |

---

## Review Criteria

- [ ] Tests follow AAA structure
- [ ] Names describe behavior, not implementation
- [ ] No real I/O in unit tests
- [ ] Each test has explicit assertions
- [ ] Tests are independent — no shared mutable state
- [ ] Error paths and edge cases are covered
- [ ] No `console.log` left in test files

---

## Implementation Notes

**Node.js / TypeScript**:
- Use Vitest (preferred) or Jest with `ts-jest`.
- Use `@faker-js/faker` for test data generation.
- Use `msw` (Mock Service Worker) for HTTP integration tests.

**Go**:
- Use `testing` package + `testify/assert` for assertions.
- Use `sqlc` + `pgx` + `dockertest` for database integration tests.
- Table-driven tests for multiple cases on same function.
