# Vite — Engineering Reference

## Best Practices

### Configuration
- Always use `defineConfig` for type-safe configuration and IDE intellisense:
  ```ts
  import { defineConfig } from 'vite';
  export default defineConfig({
    plugins: [...],
    build: { target: 'esnext' },
  });
  ```
- Use conditional config when dev and production require different plugins:
  ```ts
  export default defineConfig(({ command, mode }) => {
    const isServe = command === 'serve';
    return {
      plugins: [react(), isServe && devOnlyPlugin()].filter(Boolean),
    };
  });
  ```
- Load `.env` variables in `vite.config.ts` explicitly via `loadEnv` — they are NOT auto-injected into config:
  ```ts
  import { defineConfig, loadEnv } from 'vite';
  export default defineConfig(({ mode }) => {
    const env = loadEnv(mode, process.cwd(), '');
    return { server: { port: Number(env.APP_PORT) || 5173 } };
  });
  ```

### Environment Variables
- Prefix client-facing variables with `VITE_` — only `VITE_*` vars are exposed to the browser bundle.
- Access in code via `import.meta.env.VITE_API_URL` — never `process.env`.
- Use `import.meta.env.MODE` for environment name and `import.meta.env.DEV` / `import.meta.env.PROD` for booleans.
- Validate required env vars at app startup (e.g., with Zod) — Vite does not validate them.
- Never commit `.env.local` or `.env.production` — add them to `.gitignore`.

### Plugins
- Use `@vitejs/plugin-react` (SWC-based) for React projects:
  ```ts
  import react from '@vitejs/plugin-react-swc';
  export default defineConfig({ plugins: [react()] });
  ```
- Keep plugin order: transform plugins before framework plugins.
- Avoid plugins that run expensive transforms in dev — they defeat HMR speed.

### Build Optimization
- Code-split at route level with `React.lazy` + dynamic imports — Vite handles chunk splitting automatically.
- Set `build.target` to `'esnext'` for modern browsers or a specific version for compatibility.
- Use `build.rollupOptions.output.manualChunks` to control chunk grouping for vendor splitting:
  ```ts
  build: {
    rollupOptions: {
      output: {
        manualChunks: { vendor: ['react', 'react-dom'] },
      },
    },
  }
  ```
- Enable `build.sourcemap: true` in staging; disable in production unless needed for error tracking.

### Path Aliases
- Define aliases in `vite.config.ts` and mirror them in `tsconfig.json` — they must be in sync:
  ```ts
  resolve: {
    alias: { '@': path.resolve(__dirname, './src') },
  }
  ```
- Keep aliases minimal — deep aliasing hides module relationships.

### Dev Server
- Use `server.proxy` to forward API calls during development — avoids CORS issues:
  ```ts
  server: {
    proxy: { '/api': { target: 'http://localhost:3000', changeOrigin: true } },
  }
  ```
- Never hardcode `server.port` without a fallback — use env var with default.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Accessing `process.env.VAR` in browser code | Undefined at runtime in Vite | Use `import.meta.env.VITE_VAR` |
| Prefixing sensitive vars with `VITE_` | Exposed in client bundle | Remove `VITE_` prefix; server-only vars stay unprefixed |
| Reading `.env` vars directly in `vite.config.ts` | Not injected — silently undefined | Use `loadEnv` helper |
| No `manualChunks` for large vendor libs | One huge initial bundle | Split vendors explicitly |
| `build.target: 'es5'` | Vite/esbuild doesn't fully support ES5 | Use a polyfill pipeline or Babel for ES5 targets |
| Importing from `node_modules` inside public/ | Public files are served as-is | Import modules properly via src/ |
| Running `vite build` instead of `ionic build` in Ionic projects | Capacitor assets not copied | Use `ionic build` to trigger cap copy |

---

## Review Criteria

- [ ] `defineConfig` used — no plain object export
- [ ] `VITE_` prefix only on browser-safe variables
- [ ] `import.meta.env` used in client code — no `process.env`
- [ ] `.env.local` and `.env.*.local` in `.gitignore`
- [ ] Path aliases defined in both `vite.config.ts` and `tsconfig.json`
- [ ] Route-level code splitting with `React.lazy`
- [ ] `build.rollupOptions.manualChunks` defined for large vendors
- [ ] Dev proxy configured for API calls — no hardcoded CORS workarounds

---

## Trade-offs

**Vite vs webpack**: Vite uses native ESM in dev for near-instant HMR; webpack bundles everything. Vite is significantly faster in dev. Use webpack only for legacy projects or when an ecosystem dependency requires it.

**`@vitejs/plugin-react` vs `@vitejs/plugin-react-swc`**: The SWC variant is faster but occasionally has edge-case differences with Babel. Prefer SWC for speed; fall back to the Babel variant if you need custom Babel transforms.

**`manualChunks` vs dynamic imports**: Dynamic imports split code automatically at import boundaries. `manualChunks` gives explicit control over chunk composition. Use dynamic imports first; add `manualChunks` only when the automatic splitting produces undesirable chunk sizes.

---

## Implementation Notes

- Install: `pnpm create vite@5 my-app --template react-ts`.
- Vite 5 requires **Node 18+**.
- Config file: `vite.config.ts` at project root (auto-detected).
- Run `vite` for dev server, `vite build` for production, `vite preview` to preview the production build locally.
- For Ionic projects, use the Ionic CLI which wraps Vite: `ionic serve`, `ionic build`.
- HMR (Hot Module Replacement) is automatic — no explicit setup needed for React with `@vitejs/plugin-react`.
- Use `vite-tsconfig-paths` plugin to avoid duplicating alias config between Vite and TypeScript.
