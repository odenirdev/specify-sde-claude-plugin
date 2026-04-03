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

### Performance
- Memoize only when measured. Premature `useMemo`/`useCallback` adds noise without benefit.
- Use `React.lazy` + `Suspense` for code-split boundaries at route level.
- Virtualize long lists (TanStack Virtual, react-window) — never render 1000+ items.
- Avoid anonymous object/function creation inside JSX props when the reference matters.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `useEffect` for data fetching | Race conditions, duplication | TanStack Query |
| State that mirrors props | Sync bugs | Derive from props or lift state |
| Index as key in list | Breaks reconciliation | Use stable unique ID |
| Deeply nested prop drilling | Hard to maintain | Context or Zustand |
| `useLayoutEffect` by default | Blocks paint | Only for DOM measurement after render |
| Component does fetching AND rendering | Untestable | Separate data and view concerns |

---

## Review Criteria

- [ ] No `useEffect` for data fetching
- [ ] Query keys are stable and follow key factory pattern
- [ ] Loading and error states are handled
- [ ] Keys in lists are stable unique IDs
- [ ] No prop drilling beyond 2 levels without Context/store
- [ ] Mutations invalidate stale queries
- [ ] Large lists are virtualized

---

## Trade-offs

**Context vs Zustand**: React Context re-renders all consumers on any change. Zustand selectors prevent unnecessary re-renders. Use Context for low-frequency data (theme, auth), Zustand for high-frequency or complex client state.

**Client vs Server Components (Next.js)**: Prefer Server Components for data-heavy, no-interaction views. Client Components for interactivity. Co-locate the boundary as deep as possible.

**Formik vs React Hook Form**: Formik is simpler and more verbose. React Hook Form has better performance for large forms. For most forms, either works — be consistent within a project.

---

## Implementation Notes

- TypeScript strict mode must be enabled.
- Use absolute imports configured via `tsconfig.paths`.
- Validate environment variables at startup (e.g., with Zod).
- All async operations must have error boundaries or explicit error UI.
