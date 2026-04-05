# Turborepo — Engineering Reference

## Best Practices

### Pipeline Configuration
- Define tasks in `turbo.json` with explicit `dependsOn` to guarantee correct execution order.
- Mark tasks as `cache: false` only when they produce non-deterministic or side-effectful output (e.g., deploy, test:e2e).
- Use `outputs` in each pipeline task to scope what gets cached — omitting it caches nothing useful.
- Never rely on implicit task ordering; every dependency between packages must be declared.

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "lint": {
      "dependsOn": [],
      "outputs": []
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "cache": true
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### Caching
- Remote caching (Vercel Remote Cache or self-hosted) is required for CI efficiency — local cache only benefits one machine.
- Set `TURBO_TOKEN` and `TURBO_TEAM` env vars in CI; never hardcode them.
- Scope cache keys by environment: a `test` task running against different env vars must produce different cache entries — use `env` in pipeline config.

```json
"test": {
  "dependsOn": ["build"],
  "env": ["DATABASE_URL", "NODE_ENV"],
  "outputs": ["coverage/**"]
}
```

### Filtering and Targeting
- Use `--filter` to scope commands to one package or its dependents: `turbo build --filter=@acme/api`
- Use `--filter=...[origin/main]` in CI to build only packages changed since the last merge.
- Combine filters: `--filter=@acme/web --filter=@acme/ui` targets multiple packages.

### Package Boundaries
- Each package must be independently buildable — no cross-package relative imports outside of declared workspace dependencies.
- Declare internal packages as workspace deps in `package.json`: `"@acme/ui": "workspace:*"`.
- Shared config packages (e.g., `@acme/tsconfig`, `@acme/eslint-config`) should have no runtime dependencies.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Missing `dependsOn: ["^build"]` on a task that consumes build artifacts | Cache hits from stale outputs cause runtime failures | Always declare upstream task dependencies with `^` prefix |
| `cache: true` on a deploy task | Stale deploy step is silently skipped | Set `cache: false` on all side-effectful tasks (deploy, publish, notify) |
| Cross-package relative imports (`../../packages/ui/src`) | Breaks package boundaries; bypasses type isolation | Import via declared workspace dep (`@acme/ui`) |
| Running `turbo run build` without `--filter` in feature branches | Rebuilds everything on every PR | Use `--filter=...[origin/main]` to scope to changed packages |
| Shared `node_modules` hoisted without explicit workspace protocol | Version conflicts between packages | Use `pnpm` with `workspace:*` protocol; avoid implicit version resolution |
| `outputs` field omitted in pipeline | Turborepo caches nothing; every run rebuilds | Always declare `outputs` matching the build artifact paths |

---

## Review Criteria

- [ ] `turbo.json` `tasks` covers `build`, `lint`, `test`, and `dev` at minimum
- [ ] All tasks with upstream dependencies declare `dependsOn: ["^build"]` or equivalent
- [ ] Deploy and publish tasks have `cache: false`
- [ ] `outputs` is defined and matches actual artifact paths for every cached task
- [ ] Env vars that affect task output are listed in the task's `env` array
- [ ] Remote caching is configured via env vars — no hardcoded tokens
- [ ] Internal packages are imported as workspace deps, not relative paths
- [ ] `--filter` is used in CI to scope changed packages

---

## Trade-offs

**Turborepo vs Nx**: Turborepo has minimal config and zero opinions on project structure — ideal for existing projects. Nx enforces generators, executors, and project graph conventions — better for greenfield with strong generator discipline. Choose Turborepo when you want caching and parallelism without the Nx mental model.

**Turborepo vs manual npm/pnpm scripts**: Manual scripts provide no caching, no dependency graph, and no parallelism across packages. Turborepo adds all three with a single `turbo.json`. The cost is a new mental model for pipeline config. Worth it as soon as you have 3+ packages that need to build in order.

**Remote cache vs no remote cache**: Without remote caching, CI always rebuilds everything on cold runners. Remote caching (Vercel or self-hosted `ducktape`) can cut CI time by 60–90% on stable packages. Self-hosting adds infrastructure overhead; Vercel Remote Cache is zero-config but requires a Vercel account.

---

## Implementation Notes

- **Install**: `pnpm add turbo -D -w` at the monorepo root (the `-w` flag installs to the workspace root).
- **Entry point**: `turbo.json` lives at the monorepo root. Package-level `turbo.json` files are not supported.
- **pnpm workspace**: Pair with `pnpm-workspace.yaml` to declare package globs:
  ```yaml
  packages:
    - "packages/*"
    - "apps/*"
  ```
- **Gitignore**: Add `.turbo/` to `.gitignore` — it contains local cache artifacts.
- **CI pattern**: Run `turbo run build lint test --filter=...[origin/main]` to build only affected packages.
- **Persistent tasks**: Mark `dev` as `persistent: true` to prevent it from blocking other tasks in the pipeline.
- **Common gotcha**: If a package's `package.json` `name` field doesn't match the workspace dep name (`@acme/ui` vs `ui`), Turborepo won't link them correctly in the task graph.
