# Template Mapping — init scope

When running `init`, read each template file from `references/` and write the filled version to the consuming project. Fill with real data extracted from the codebase and `./.specify/specs`. Remove placeholder sections that have no data yet — never leave template boilerplate in the output.

| Template (this references/ folder) | Target (consuming project) | Purpose |
|---|---|---|
| `references/template-index.md` | `./.specify/docs/index.md` | Project overview, stack, integrations, features, agents |
| `references/template-architecture.md` | `./.specify/docs/architecture.md` | Component map, data flow, ADRs, constraints |
| `references/template-integrations.md` | `./.specify/docs/integrations.md` | External services, auth, retry, failure behavior |
| `references/template-operations.md` | `./.specify/docs/operations.md` | Environments, config, deploy, health checks, runbooks |
| `references/template-claude-root.md` | `./CLAUDE.md` | Root context bridge for agent runtimes; must stay minimal |
| `references/template-readme-managed.md` | `./README.md` managed block `<!-- docs-sync:start --> ... <!-- docs-sync:end -->` | Short human overview, getting started, commands, and links |
| `references/template-claude-package.md` | `./<package>/CLAUDE.md` | Package-level context bridge for monorepos; create only when local `./.specify/docs/` exists |
| — | `./.specify/docs/decisions/` | Directory for ADR files (create empty) |

## Notes

- Read each template file before writing the target — understand its sections first.
- Only create target files that don't already exist; if a file exists, skip it during `init` (use `index`, `architecture`, `integrations`, `claude`, or `readme` scopes to update existing files).
- For `README.md`, update only the content between `<!-- docs-sync:start -->` and `<!-- docs-sync:end -->`. Preserve all content outside that block.
- For `CLAUDE.md`, keep the file link-oriented and minimal. It must route to `./.specify/docs` instead of copying rules or architecture details.
- In monorepo mode, create package-level `CLAUDE.md` files only for packages with local `./.specify/docs/`.
- `references/template-conventions.md` is optional — create `./.specify/docs/conventions.md` only if the consuming project has explicit naming or architecture conventions documented somewhere.
