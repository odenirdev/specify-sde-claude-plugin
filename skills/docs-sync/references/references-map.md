# References File Mapping

Load all matching reference files before writing documentation. These inform how you interpret the codebase and what you document about it.

| Stack Signal | Reference File |
|---|---|
| TypeScript | `references/languages/typescript.md` |
| Go | `references/languages/go.md` |
| NestJS | `references/frameworks/nestjs.md` |
| React / Next.js | `references/frameworks/react.md` |
| Ionic / @ionic/react | `references/frameworks/ionic-react.md` |
| Capacitor / @capacitor | `references/frameworks/capacitor.md` |
| react-router-dom | `references/libs/react-router-dom.md` |
| Prisma | `references/libs/prisma.md` |
| Axios | `references/libs/axios.md` |
| Vite / vite.config | `references/libs/vite.md` |
| ionicons | `references/libs/ionicons.md` |
| `turbo.json` / turbo | `references/libs/turborepo.md` |
| Any monorepo (`pnpm-workspace.yaml` / `workspaces` in `package.json`) | `references/practices/monorepo.md` |
| `@langchain/langgraph` | `references/frameworks/langgraph.md` |
| `@langchain/core` | `references/libs/langchain-core.md` |
| `@langchain/groq` | `references/libs/langchain-groq.md` |
| Serverless Framework (`serverless` dep or `serverless.yml`) | `references/frameworks/serverless-framework.md` |
| `serverless.yml` / AWS Lambda | `references/platforms/aws-lambda.md` |
| Any | `references/practices/hexagonal-architecture.md` |
| Any | `references/utilities/error-handling.md` |
| Any | `references/practices/documentation-derivation.md` |

## References Authoring

To add or update a reference file, edit `references/<category>/<name>.md` directly and follow the established structure already used across the repository:

1. `## Best Practices`
2. `## Anti-Patterns`
3. `## Review Criteria`
4. `## Trade-offs`
5. `## Implementation Notes`

Use a nearby file in the same category as the concrete pattern, keep the content opinionated and reusable, and update the mapping table above when the new reference should be auto-loaded by `docs-sync`.
