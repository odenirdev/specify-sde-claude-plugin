# Monorepo — Engineering Reference

## Best Practices

### Package Boundaries
- Each package in the monorepo must be independently buildable and testable.
- Cross-package imports must go through declared workspace dependencies — never via relative paths that escape the package root.
- Shared packages (UI, config, utils) must have explicit, narrow APIs — do not leak internal implementation details.
- If a type or function is needed by more than one package, promote it to a shared package explicitly; do not copy-paste.

### Workspace Structure
- Group packages by role, not by technology: `apps/` for deployable applications, `packages/` for shared libraries.
- Shared config packages (TypeScript config, ESLint config, Prettier config) belong in `packages/` with no runtime dependencies.
- Keep each package's `package.json` `name` field namespaced: `@acme/api`, `@acme/web`, `@acme/ui`.

```
<monorepo-root>/
  apps/
    web/               # Next.js or React app
    api/               # NestJS or Express backend
  packages/
    ui/                # Shared component library
    tsconfig/          # Shared TypeScript config
    eslint-config/     # Shared ESLint config
  pnpm-workspace.yaml
  turbo.json
  package.json
```

### Dependency Management
- Use `pnpm` with the `workspace:*` protocol for internal deps — it ensures the local version is always used, not a published one.
- Pin external dependency versions at the root; let individual packages inherit via `catalog:` or root `devDependencies` for tooling.
- Never install the same dependency at different major versions across packages without a documented reason.

### `.specify` Convention for Monorepos

This is the canonical layout for `specify-sde` specs and docs in a monorepo:

```
<monorepo-root>/
  .specify/
    specs/             ← ONLY place for specs (all features and bugs for the entire monorepo)
    docs/              ← docs for the monorepo as a whole (index, architecture, stack)
  apps/
    web/
      .specify/
        docs/          ← docs specific to the web app package
        # NEVER create specs/ here
    api/
      .specify/
        docs/          ← docs specific to the api package
        # NEVER create specs/ here
  packages/
    ui/
      .specify/
        docs/          ← docs specific to the ui package
        # NEVER create specs/ here
```

**Rules:**
- `.specify/specs/` exists **only at the monorepo root**. Every feature spec, bug report, and task breakdown lives here, regardless of which package it affects.
- `.specify/docs/` exists at the root **and** inside each package that needs package-level documentation.
- Package-level `.specify/docs/` documents only what is specific to that package: its stack, its architecture, its public API contract.
- Root `.specify/docs/` documents the monorepo as a whole: overall architecture, cross-cutting concerns, ADRs, integrations.
- **Never** create a `specs/` directory inside a package — it breaks the single source of truth principle and makes cross-package features impossible to track atomically.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `specs/` inside individual packages | Specs fragmented across the repo; cross-package features require multiple spec files | Move all specs to monorepo root `.specify/specs/` |
| Root `.specify/docs/` documents package internals | Root docs become package-specific and stale | Keep root docs at architecture level; delegate package details to package-level docs |
| Relative imports across package boundaries (`../../packages/ui/src`) | Bypasses package isolation; breaks when packages move | Declare workspace dep and import by package name |
| Shared mutable package with no versioning strategy | Breaking changes to shared package break all consumers silently | Use `workspace:*` and treat shared package changes as PRs that touch all affected packages |
| Running specs for one package without referencing monorepo root | Feature context lost; no cross-package dependency visible | Always write specs at the root; use the `scope` field in spec frontmatter to indicate the affected package(s) |
| `docs-sync` run inside a package without monorepo awareness | Incorrect stack detection; docs duplicate instead of complement root docs | Always run `docs-sync` in monorepo mode from the root — it will write per-package docs automatically |

---

## Review Criteria

- [ ] All specs live under `<root>/.specify/specs/` — no `specs/` directories in packages
- [ ] Each package has at most a `.specify/docs/` directory — never `specs/`
- [ ] Cross-package imports use workspace protocol (`@acme/ui`), not relative paths
- [ ] Shared packages have namespaced `name` fields (`@acme/*`)
- [ ] Root `turbo.json` declares all tasks with correct `dependsOn`
- [ ] Root `.specify/docs/` covers architecture and cross-cutting concerns only
- [ ] Package-level `.specify/docs/` covers only that package's internals

---

## Trade-offs

**Monorepo vs polyrepo**: Monorepo provides atomic commits across packages, shared tooling, and easier refactoring of shared code. Polyrepo provides stronger isolation, independent deploy cadence, and smaller blast radius per change. Choose monorepo when packages share code frequently or must be versioned together. Choose polyrepo when packages have separate teams, SLAs, or deployment pipelines with no shared code.

**Single `specs/` root vs per-package specs**: Single root maintains one source of truth and allows atomic feature specs that span multiple packages. Per-package specs seem simpler initially but make cross-package features ambiguous and create duplication. The single root is the correct convention for this plugin.

**pnpm workspaces vs npm/yarn workspaces**: pnpm workspaces use a content-addressable store and strict symlink resolution — no phantom dependencies (packages imported without being listed in `package.json`). npm/yarn workspaces hoist all deps, which allows phantom imports that break in isolation. Prefer pnpm for correctness.

---

## Implementation Notes

- **`pnpm-workspace.yaml`** at the root declares which directories are packages:
  ```yaml
  packages:
    - "apps/*"
    - "packages/*"
  ```
- **Root `package.json`** should have `"private": true` — the root is never published.
- **`docs-sync` in monorepo mode**: run from the monorepo root. The skill detects workspace config and writes per-package docs automatically. Specs are only written at the root.
- **Spec frontmatter for monorepos**: include a `scope` field to indicate which packages are affected:
  ```markdown
  ---
  scope: [apps/web, packages/ui]
  ---
  ```
- **ADRs**: architecture decisions that affect multiple packages belong in root `.specify/docs/decisions/`. Package-specific decisions can go in `./<package>/.specify/docs/decisions/`.
- **Common gotcha**: running `docs-sync` inside a package directory (e.g., `cd apps/web && docs-sync`) without monorepo context will produce incomplete stack detection. Always run from the monorepo root.
