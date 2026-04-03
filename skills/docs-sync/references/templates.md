# Template Mapping — init scope

When running `init`, read each template file from `references/` and write the filled version to the consuming project. Fill with real data extracted from the codebase and `./.specify/specs`. Remove placeholder sections that have no data yet — never leave template boilerplate in the output.

| Template (this references/ folder) | Target (consuming project) | Purpose |
|---|---|---|
| `references/template-index.md` | `./.specify/docs/index.md` | Project overview, stack, integrations, features, agents |
| `references/template-architecture.md` | `./.specify/docs/architecture.md` | Component map, data flow, ADRs, constraints |
| `references/template-integrations.md` | `./.specify/docs/integrations.md` | External services, auth, retry, failure behavior |
| `references/template-operations.md` | `./.specify/docs/operations.md` | Environments, config, deploy, health checks, runbooks |
| — | `./.specify/docs/decisions/` | Directory for ADR files (create empty) |

## Notes

- Read each template file before writing the target — understand its sections first.
- Only create target files that don't already exist; if a file exists, skip it during `init` (use `index`, `architecture`, or `integrations` scopes to update existing files).
- `references/template-conventions.md` is optional — create `./.specify/docs/conventions.md` only if the consuming project has explicit naming or architecture conventions documented somewhere.
