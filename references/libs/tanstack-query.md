# TanStack Query (React Query) — Engineering Reference

## Best Practices

### Client Setup
- Create a single `QueryClient` at the app root and provide it via `QueryClientProvider`. Never instantiate the client inside a component render.
- Set intentional defaults for `staleTime`, `retry`, and `refetchOnWindowFocus` based on product UX — especially in mobile/web hybrid apps where focus changes can be noisy.
- Keep React Query focused on **server state**. Continue using local component state for ephemeral UI state such as modals, tabs, and input drafts.

### Queries
- Use stable, structured query keys (prefer arrays) instead of ad-hoc strings:
  ```ts
  ['health']
  ['deck-analysis', battlelogId]
  ['player', playerTag, 'matches']
  ```
- Keep each `queryFn` small, side-effect free, and responsible only for fetching/parsing data.
- Use `enabled` for dependent queries rather than conditional hook execution:
  ```ts
  useQuery({
    queryKey: ['deck-analysis', battlelogId],
    queryFn: () => getDeckAnalysis(battlelogId),
    enabled: Boolean(battlelogId),
  });
  ```
- Use `select` to shape response data for the UI instead of copying query data into local `useState`.

### Mutations
- Use `useMutation` for writes and submissions — never `useQuery` for POST/PUT/DELETE flows.
- Invalidate or update the relevant query keys after successful mutations so cached data stays fresh.
- Prefer optimistic updates only when the rollback path is well understood and tested.

### Error & Loading States
- Render explicit loading, error, and empty states from query status (`isLoading`, `isFetching`, `isError`).
- Map HTTP/client errors into user-safe messages at the API adapter layer before they reach the component.
- Keep retries intentional. Automatic retries are useful for transient failures, but can hurt UX on validation/auth errors.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| New `QueryClient()` inside component render | Cache is recreated, queries refetch constantly | Create once at app bootstrap |
| String-only, inconsistent query keys | Cache collisions and invalidation bugs | Use structured array keys |
| Copying `query.data` into `useState` | Duplicated server state, stale UI | Derive UI directly from query result |
| Using `useQuery` for form submit/write flows | Incorrect lifecycle semantics | Use `useMutation` |
| Blind `invalidateQueries()` on everything | Excess network load and noisy UI | Invalidate only related keys |
| Disabling all refetch/retry defaults without analysis | Hidden stale-data and resiliency issues | Tune defaults intentionally |

---

## Review Criteria

- [ ] App root provides a single `QueryClientProvider`
- [ ] Query keys are stable, descriptive, and array-based
- [ ] `queryFn` contains fetch logic only — no UI side effects
- [ ] Dependent queries use `enabled`, not conditional hooks
- [ ] Writes use `useMutation` and refresh related queries explicitly
- [ ] Components show clear loading/error/empty states
- [ ] Server state is not duplicated into local `useState` without need

---

## Trade-offs

**TanStack Query vs `useEffect` + `fetch`**: direct hooks are fine for one-off calls, but they scale poorly when caching, refetching, retries, invalidation, and background freshness matter. TanStack Query adds structure and cache management for server state.

**TanStack Query vs global client-state stores**: Query excels at remote/server state, not local UI orchestration. Keep transient UI state in React local state or a dedicated client-state store.

---

## Implementation Notes

- Current package name is `@tanstack/react-query`; older docs and examples may still show the legacy v3 import path `react-query`.
- For hybrid apps (Ionic + Capacitor), review focus/refetch behavior so app foreground transitions do not trigger surprising reloads.
- Pair with schema validation (for example `zod`) at the API boundary to ensure cached data has a trusted runtime shape.
