# React — Engineering Reference

## Best Practices

### Component Design
- One component = one responsibility. A component that fetches, transforms, and renders data needs to be split.
- Co-locate state as close to usage as possible. Lift only when genuinely shared.
- Prefer composition over configuration. Pass children and render props instead of a proliferation of boolean flags.
- Keep components small: if a component's JSX exceeds ~80 lines, extract parts.

### State Management
- Local UI state (open/closed, form input) → `useState`
- Derived state → useMemo or compute inline, not stored in state
- Async server state → React Query / TanStack Query (not `useEffect + useState`)
- Global client state (auth, preferences) → Zustand or Context with a stable reference
- Never sync server data into local state — it creates two sources of truth

### Concurrent Features (React 18)
- Wrap non-urgent state updates in `startTransition` to keep the UI responsive:
  ```tsx
  startTransition(() => setSearchQuery(input));
  ```
- Use `useTransition` when the component needs to show a pending indicator:
  ```tsx
  const [isPending, startTransition] = useTransition();
  ```
- Use `useDeferredValue` to defer re-rendering an expensive derived value:
  ```tsx
  const deferredQuery = useDeferredValue(query); // renders with stale value during fast typing
  ```
- Do not use `startTransition` for urgent updates (keypresses, direct input) — only for non-urgent UI changes.

### Root API (React 18)
- Always use `createRoot` — never use the legacy `ReactDOM.render`:
  ```tsx
  import { createRoot } from 'react-dom/client';
  createRoot(document.getElementById('root')!).render(<App />);
  ```
- For hydration (SSR), use `hydrateRoot` instead of `ReactDOM.hydrate`.

### Data Fetching
- Use TanStack Query for all server data. Do not write fetch logic inside `useEffect`.
- Define query key factories as constants:
  ```ts
  export const userKeys = {
    all: ['users'] as const,
    detail: (id: string) => ['users', id] as const,
  };
  ```
- Mutations must invalidate or update relevant queries on success.
- Always handle loading and error states explicitly — never assume data is available.

### Hooks
- Custom hooks must start with `use` and encapsulate a single concern.
- Never call hooks conditionally.
- `useEffect` is for synchronizing with external systems (DOM, WebSocket, timers). For data fetching, use React Query.
- Always clean up effects: clear timers, unsubscribe, abort requests.
- Use `useId` (React 18) for stable, SSR-safe IDs — never generate random IDs during render:
  ```tsx
  const id = useId(); // stable across server and client
  ```
- Use `useSyncExternalStore` (React 18) to subscribe to external stores instead of manual `useEffect + useState`:
  ```tsx
  const snapshot = useSyncExternalStore(store.subscribe, store.getSnapshot);
  ```

### Performance
- Memoize only when measured. Premature `useMemo`/`useCallback` adds noise without benefit.
- React 18 with automatic batching groups all state updates (including inside timeouts and promises) — do not rely on multiple renders from synchronous calls.
- Use `React.lazy` + `Suspense` for code-split boundaries at route level.
- Virtualize long lists (TanStack Virtual, react-window) — never render 1000+ items.
- Avoid anonymous object/function creation inside JSX props when the reference matters.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `useEffect` for data fetching | Race conditions, duplication | TanStack Query |
| `ReactDOM.render` (legacy API) | Opt-out from concurrent features | Use `createRoot` |
| State that mirrors props | Sync bugs | Derive from props or lift state |
| Index as key in list | Breaks reconciliation | Use stable unique ID |
| Deeply nested prop drilling | Hard to maintain | Context or Zustand |
| `useLayoutEffect` by default | Blocks paint | Only for DOM measurement after render |
| Component does fetching AND rendering | Untestable | Separate data and view concerns |
| Generating IDs with `Math.random()` in render | Hydration mismatch (SSR) | Use `useId` |
| Direct mutation of state | Silent bugs, missed re-renders | Always produce new references |

---

## Review Criteria

- [ ] `createRoot` used — no legacy `ReactDOM.render`
- [ ] No `useEffect` for data fetching
- [ ] Query keys are stable and follow key factory pattern
- [ ] Loading and error states are handled
- [ ] Keys in lists are stable unique IDs
- [ ] No prop drilling beyond 2 levels without Context/store
- [ ] Mutations invalidate stale queries
- [ ] Large lists are virtualized
- [ ] `useTransition`/`startTransition` used for non-urgent updates
- [ ] `useId` used for stable IDs — no `Math.random()` in render

---

## Trade-offs

**Context vs Zustand**: React Context re-renders all consumers on any change. Zustand selectors prevent unnecessary re-renders. Use Context for low-frequency data (theme, auth), Zustand for high-frequency or complex client state.

**`useTransition` vs `useDeferredValue`**: `useTransition` wraps the update trigger and requires access to the setter. `useDeferredValue` wraps the value and works when you can't change the update source. Prefer `useTransition` when you control the update; `useDeferredValue` for third-party or prop-driven values.

**Client vs Server Components (Next.js)**: Prefer Server Components for data-heavy, no-interaction views. Client Components for interactivity. Co-locate the boundary as deep as possible.

**Formik vs React Hook Form**: Formik is simpler and more verbose. React Hook Form has better performance for large forms. For most forms, either works — be consistent within a project.

---

## Implementation Notes

- TypeScript strict mode must be enabled.
- Use absolute imports configured via `tsconfig.paths`.
- Validate environment variables at startup (e.g., with Zod).
- All async operations must have error boundaries or explicit error UI.
- Automatic batching in React 18 means `flushSync` is needed when you explicitly need synchronous DOM updates after state changes.
- Install: `pnpm add react@18 react-dom@18 && pnpm add -D @types/react@18 @types/react-dom@18`.
