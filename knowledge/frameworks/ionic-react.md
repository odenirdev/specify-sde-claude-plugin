# Ionic React — Engineering Reference

## Best Practices

### Component Structure
- Use Ionic UI components (`IonPage`, `IonHeader`, `IonContent`, `IonFooter`) as page shells — never build custom page chrome.
- Every routable view must be wrapped in `<IonPage>` — Ionic's animation and lifecycle system depends on it.
- Keep `<IonContent>` as the scrollable container; do not add `overflow: auto` to arbitrary wrappers.
- Lazy-load pages with `React.lazy` — Ionic renders pages off-screen for animation, so large bundles degrade transition performance.

### Routing with IonRouterOutlet
- Use `<IonRouterOutlet>` (not plain `<Switch>`) to enable native-like page transitions and stack management.
- Nest tab routes inside `<IonTabs>` + `<IonTabBar>` + `<IonRouterOutlet>` — this is the only supported tab routing pattern.
- Do not render pages outside `IonRouterOutlet` — doing so bypasses animation and lifecycle hooks.
- Keep route definitions flat: deeply nested dynamic routes produce animation glitches.

### Lifecycle Hooks
- Use Ionic lifecycle hooks for side effects tied to page visibility — not just React's `useEffect`:
  - `ionViewWillEnter` — triggered every time the page becomes visible (use for refreshing data)
  - `ionViewDidEnter` — page fully visible, safe to focus elements
  - `ionViewWillLeave` — cleanup before page leaves
  - `ionViewDidLeave` — page fully gone
  ```tsx
  import { useIonViewWillEnter } from '@ionic/react';
  useIonViewWillEnter(() => { refetch(); });
  ```
- Do not rely solely on `useEffect` with empty deps for page-entry logic — Ionic keeps pages alive in the stack, so `useEffect` runs only on mount, not on every navigation.

### Theming and Platform
- Use CSS custom properties for theming — never override Ionic component internals with deep selectors.
- Use `isPlatform('ios' | 'android' | 'hybrid' | 'desktop')` for platform-specific logic — never `navigator.userAgent` parsing.
- Apply `mode="ios"` or `mode="md"` explicitly when a consistent cross-platform look is required; otherwise default is platform-adaptive.

### Forms and Inputs
- Use `IonInput`, `IonTextarea`, `IonSelect` instead of native HTML inputs — native elements break Ionic's touch/scroll handling on mobile.
- Always bind `onIonChange` (not `onChange`) for Ionic form components — `onChange` fires on every keystroke, `onIonChange` fires on commit.
- Use `IonItem` + `IonLabel` wrappers for all form fields to get consistent mobile styling.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Rendering pages outside `<IonRouterOutlet>` | Bypasses animations and lifecycle | Wrap all routes in `IonRouterOutlet` |
| Using `useEffect(fn, [])` for page-entry data refresh | Runs only on mount, not on return navigation | Use `useIonViewWillEnter` |
| Using `<Switch>` instead of `<IonRouterOutlet>` | No native transitions | Replace with `IonRouterOutlet` |
| Overriding Ionic component internals with CSS | Breaks across Ionic versions | Use CSS custom properties / variables |
| Using native `<input>` inside `IonItem` | Touch/scroll conflicts on mobile | Use `IonInput` or `IonTextarea` |
| Importing all of `@ionic/react` | Inflates bundle | Import only used components |
| Using `onChange` on `IonInput` | Fires on every keystroke, perf issues | Use `onIonChange` |

---

## Review Criteria

- [ ] Every routable view wrapped in `<IonPage>`
- [ ] All routes rendered inside `<IonRouterOutlet>` (not `<Switch>`)
- [ ] Page-entry data refresh uses `useIonViewWillEnter`, not `useEffect(fn, [])`
- [ ] Platform detection uses `isPlatform()`, not user-agent parsing
- [ ] Ionic form components (`IonInput`, `IonSelect`) used instead of native inputs
- [ ] `onIonChange` used on Ionic inputs, not `onChange`
- [ ] Theming done via CSS custom properties, not deep CSS overrides
- [ ] Heavy pages lazy-loaded with `React.lazy`

---

## Trade-offs

**Ionic React vs React Native**: Ionic renders web components in a WebView; React Native renders native widgets. Ionic ships faster and shares more web code, but RN delivers better native feel and access. Choose Ionic when the team is web-first and time-to-market matters; RN when native fidelity is critical.

**`IonRouterOutlet` vs standard React Router**: `IonRouterOutlet` enables the page stack, back gesture, and view caching that define native-feel apps. Dropping it means losing all Ionic animations. Always use `IonRouterOutlet` when building with Ionic.

**Global `mode` prop (iOS vs MD)**: Using one mode globally gives consistency but loses platform-adaptive behavior. Use platform-adaptive defaults unless a unified brand look is a hard requirement.

---

## Implementation Notes

- Install: `pnpm add @ionic/react @ionic/react-router ionicons`.
- Setup in `main.tsx`: call `setupIonicReact()` before rendering and import `@ionic/react/css/core.css` + theme variables.
- Add `IonApp` as the root wrapper — it applies safe area insets and platform class names.
- Use `@capacitor/status-bar` and `@capacitor/safe-area` for notch/safe-area handling in production builds.
- Ionic 8 requires React 18 — ensure `createRoot` is used (not `ReactDOM.render`).
- Run `ionic build` (not plain `vite build`) for production to ensure Capacitor assets are copied correctly.
