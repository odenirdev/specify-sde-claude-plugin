# React Router DOM v5 — Engineering Reference

## Best Practices

### Router Setup
- Wrap the entire app in a single `<BrowserRouter>` — never nest multiple routers.
- Use `<HashRouter>` only for static file hosting without server-side redirect support.
- Define routes inside `<Switch>` so only the first match renders:
  ```tsx
  <Switch>
    <Route exact path="/" component={Home} />
    <Route path="/users/:id" component={UserDetail} />
    <Route component={NotFound} />
  </Switch>
  ```
- Always use `exact` on root `/` routes — without it, every path matches `/`.

### Route Matching
- Use `exact` to prevent partial matches on parent paths.
- Order routes from most-specific to least-specific inside `<Switch>`.
- Use `:param` for dynamic segments and access them via `useParams()`:
  ```tsx
  const { id } = useParams<{ id: string }>();
  ```
- Use `useRouteMatch` to build nested route paths without hardcoding the parent prefix:
  ```tsx
  const { path, url } = useRouteMatch();
  // <Route path={`${path}/settings`} />
  // <Link to={`${url}/settings`} />
  ```

### Navigation Hooks (v5.1+)
- Use `useHistory` to programmatically navigate — never use `window.location`:
  ```tsx
  const history = useHistory();
  history.push('/dashboard');
  history.replace('/login'); // no back stack entry
  ```
- Use `useLocation` to read the current URL (pathname, search, hash, state):
  ```tsx
  const location = useLocation<{ from: string }>();
  ```
- Use `useParams` for URL parameters — do not read params from `match` prop.

### Protected Routes
- Implement auth guards as a wrapper component using `<Redirect>`:
  ```tsx
  function PrivateRoute({ component: Component, ...rest }) {
    const isAuthenticated = useAuth();
    return (
      <Route
        {...rest}
        render={({ location }) =>
          isAuthenticated
            ? <Component />
            : <Redirect to={{ pathname: '/login', state: { from: location } }} />
        }
      />
    );
  }
  ```
- Pass `location` in redirect state so login can return the user to the original page.

### Links
- Use `<Link>` for internal navigation — never `<a href>`.
- Use `<NavLink>` with `activeClassName` or `activeStyle` for nav menus.
- Pass objects to `to` when you need state or query params:
  ```tsx
  <Link to={{ pathname: '/search', search: '?q=foo', state: { from: 'home' } }}>Search</Link>
  ```

### Nested Routes
- Define nested routes inside the child component (not at the top level) using `useRouteMatch`:
  ```tsx
  function UserPage() {
    const { path } = useRouteMatch();
    return (
      <Switch>
        <Route exact path={path} component={UserProfile} />
        <Route path={`${path}/settings`} component={UserSettings} />
      </Switch>
    );
  }
  ```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Multiple `<BrowserRouter>` wrappers | Context conflicts, broken navigation | Single router at app root |
| Missing `exact` on root `/` route | Every path matches, wrong component renders | Add `exact` to `/` |
| `window.location.href` for navigation | Breaks SPA, full page reload | Use `useHistory().push()` |
| Routes outside `<Switch>` | Multiple components render for the same URL | Wrap routes in `<Switch>` |
| Hardcoding parent path in nested routes | Breaks when parent path changes | Use `useRouteMatch().path` |
| Reading params from `match` prop | Verbose and requires prop threading | Use `useParams()` hook |
| `<a href>` for internal links | Full page reload, loses SPA state | Use `<Link>` |

---

## Review Criteria

- [ ] Single `<BrowserRouter>` at app root
- [ ] Routes wrapped in `<Switch>`, ordered from most to least specific
- [ ] Root `/` route has `exact`
- [ ] All programmatic navigation uses `useHistory()`, not `window.location`
- [ ] Protected routes use `<Redirect>` with `state: { from: location }`
- [ ] Nested routes use `useRouteMatch().path` for path composition
- [ ] URL parameters accessed via `useParams`, not `match.params`
- [ ] Internal links use `<Link>` or `<NavLink>`, never `<a>`

---

## Trade-offs

**React Router v5 vs v6**: v6 is the current major version — it replaces `<Switch>` with `<Routes>`, `useHistory` with `useNavigate`, and introduces relative route matching. If on v5, avoid v6 patterns; if starting a new project, use v6 unless Ionic React or another lib requires v5.

**`component` prop vs `render` prop**: Use `component` for simple cases. Use `render` when you need to pass additional props or access route props (match, location, history) without extra wrapper components.

**BrowserRouter vs MemoryRouter in tests**: `MemoryRouter` in tests avoids browser API dependency — wrap components under test in `MemoryRouter` with `initialEntries`.

---

## Implementation Notes

- Install: `pnpm add react-router-dom@5 && pnpm add -D @types/react-router-dom`.
- In TypeScript, type `useParams` with a generic: `useParams<{ id: string }>()`.
- Access query strings via `useLocation().search` + `new URLSearchParams(search)`.
- `<Redirect>` used inside `<Switch>` is matched like any other route — order matters.
- Ionic React 8 requires React Router DOM v5 (not v6) — do not upgrade without checking Ionic compatibility.
- `useHistory` is removed in v6 — keep this in mind if reading v6 docs by mistake.
