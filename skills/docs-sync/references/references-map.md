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

To add or update a reference file, use the `references-update` skill. Templates for each category live in:

| Category | Template |
|---|---|
| frameworks | `skills/references-update/references/framework-template.md` |
| languages | `skills/references-update/references/language-template.md` |
| libs | `skills/references-update/references/lib-template.md` |
| utilities | `skills/references-update/references/utility-template.md` |
| practices | `skills/references-update/references/practice-template.md` |
