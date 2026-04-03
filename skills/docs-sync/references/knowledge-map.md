# Knowledge File Mapping

Load all matching knowledge files before writing documentation. These inform how you interpret the codebase and what you document about it.

| Stack Signal | Knowledge File |
|---|---|
| TypeScript | `knowledge/languages/typescript.md` |
| Go | `knowledge/languages/go.md` |
| NestJS | `knowledge/frameworks/nestjs.md` |
| React / Next.js | `knowledge/frameworks/react.md` |
| Ionic / @ionic/react | `knowledge/frameworks/ionic-react.md` |
| Capacitor / @capacitor | `knowledge/frameworks/capacitor.md` |
| react-router-dom | `knowledge/libs/react-router-dom.md` |
| Prisma | `knowledge/libs/prisma.md` |
| Axios | `knowledge/libs/axios.md` |
| Vite / vite.config | `knowledge/libs/vite.md` |
| ionicons | `knowledge/libs/ionicons.md` |
| Any | `knowledge/practices/hexagonal-architecture.md` |
| Any | `knowledge/utilities/error-handling.md` |
| Any | `knowledge/practices/documentation-derivation.md` |

## Knowledge Authoring

To add or update a knowledge file, use the `knowledge-update` skill. Templates for each category live in:

| Category | Template |
|---|---|
| frameworks | `skills/knowledge-update/references/framework-template.md` |
| languages | `skills/knowledge-update/references/language-template.md` |
| libs | `skills/knowledge-update/references/lib-template.md` |
| utilities | `skills/knowledge-update/references/utility-template.md` |
| practices | `skills/knowledge-update/references/practice-template.md` |
