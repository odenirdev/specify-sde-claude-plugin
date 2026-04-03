# Stack Detection Signals

## Files to Read

| File | Stack Signal |
|---|---|
| `package.json` | Node.js, dependencies, devDependencies |
| `go.mod` | Go modules |
| `pom.xml` | Java/Maven |
| `build.gradle` | Java/Gradle |
| `requirements.txt` | Python |
| `pyproject.toml` | Python (uv/poetry) |
| `Cargo.toml` | Rust |

## Node.js Dependency Signals

For Node.js projects, inspect `dependencies` and `devDependencies` in `package.json`:

| Dependency | Stack Signal |
|---|---|
| `typescript` | TypeScript |
| `@nestjs/core` | NestJS |
| `react`, `react-dom` | React |
| `next` | Next.js |
| `@prisma/client` | Prisma |
| `axios` | Axios |
| `fastify` | Fastify |
| `express` | Express |
| `zod` | Zod validation |
| `@tanstack/react-query` | TanStack Query |
