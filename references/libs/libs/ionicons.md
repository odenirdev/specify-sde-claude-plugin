# Ionicons — Engineering Reference

## Best Practices

### Installation and Setup
- In Ionic React projects, Ionicons is included via `@ionic/react` — no separate install needed.
- For non-Ionic React projects, import the web component scripts in `index.html`:
  ```html
  <script type="module" src="https://unpkg.com/ionicons@7/dist/ionicons/ionicons.esm.js"></script>
  <script nomodule src="https://unpkg.com/ionicons@7/dist/ionicons/ionicons.js"></script>
  ```
- In Ionic React, use the `<IonIcon>` component (not the raw `<ion-icon>` web component) for proper React integration:
  ```tsx
  import { IonIcon } from '@ionic/react';
  import { heartOutline } from 'ionicons/icons';
  <IonIcon icon={heartOutline} />
  ```

### Icon Variants
- Each icon has three variants — choose based on platform convention:
  - **Filled** (default): `heart` — used for selected or active states
  - **Outline**: `heartOutline` — used for inactive states or MD design
  - **Sharp**: `heartSharp` — used for sharp corners, alternative style
  ```tsx
  import { heart, heartOutline, heartSharp } from 'ionicons/icons';
  <IonIcon icon={heart} />          // filled
  <IonIcon icon={heartOutline} />   // outline
  <IonIcon icon={heartSharp} />     // sharp
  ```
- Match the variant to the platform: `ios` mode typically uses outline; `md` mode uses filled.

### Tree-Shaking (CRITICAL)
- Always import icon constants from `ionicons/icons`, not the icon name string:
  ```tsx
  // BAD — loads all icons, no tree-shaking
  <IonIcon name="heart" />

  // GOOD — only this icon is bundled
  import { heartOutline } from 'ionicons/icons';
  <IonIcon icon={heartOutline} />
  ```
- Never use the `name` prop on `<IonIcon>` in build environments — it bypasses tree-shaking and inflates the bundle.

### Sizing
- Use the `size` prop for standard sizes: `"small"` or `"large"`.
- For custom sizes, use the CSS `font-size` property on the component — prefer multiples of 8px:
  ```tsx
  <IonIcon icon={heartOutline} style={{ fontSize: '32px' }} />
  ```
- Icon color inherits from CSS `color` property — no separate color prop needed:
  ```tsx
  <IonIcon icon={heartOutline} style={{ color: 'var(--ion-color-primary)' }} />
  ```

### Accessibility
- Decorative icons (visual only): add `aria-hidden="true"`:
  ```tsx
  <IonIcon icon={heartOutline} aria-hidden="true" />
  ```
- Interactive standalone icons (no surrounding label): add `aria-label`:
  ```tsx
  <IonIcon icon={closeOutline} aria-label="Close dialog" />
  ```
- Icons inside labeled buttons or links: hide the icon and label the parent:
  ```tsx
  <IonButton aria-label="Add to favorites">
    <IonIcon icon={heartOutline} aria-hidden="true" slot="icon-only" />
  </IonButton>
  ```

### Custom Icons
- Use the `src` prop to load custom SVGs from the `public/` directory:
  ```tsx
  <IonIcon src="/icons/custom-logo.svg" />
  ```
- Custom SVGs must be clean: no `<script>` tags, no event handlers, no external references.
- Adjust outline weight of custom icons with the CSS custom property:
  ```css
  ion-icon { --ionicon-stroke-width: 24px; } /* default is 32px */
  ```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `<IonIcon name="heart" />` in build | No tree-shaking, entire icon set bundled | Import from `ionicons/icons` and use `icon` prop |
| Decorative icons without `aria-hidden` | Screen readers announce meaningless icons | Add `aria-hidden="true"` |
| Interactive icon without label | Screen readers cannot describe the action | Add `aria-label` or label the parent |
| Hardcoded pixel color on icon | Breaks dark mode and theming | Use CSS `color: var(--ion-color-primary)` |
| Custom SVG with inline scripts | Sanitized/blocked by Ionic | Remove scripts from SVG |
| Using `<ion-icon>` web component directly in React | No React integration, loses TypeScript types | Use `<IonIcon>` from `@ionic/react` |

---

## Review Criteria

- [ ] Icons imported as constants from `ionicons/icons`, not used as name strings
- [ ] `icon` prop used on `<IonIcon>`, not `name`
- [ ] Decorative icons have `aria-hidden="true"`
- [ ] Interactive standalone icons have `aria-label`
- [ ] Icon colors use CSS variables or inherited color — no hardcoded hex
- [ ] Custom SVGs are free of scripts and event handlers
- [ ] Icon sizes use multiples of 8px

---

## Trade-offs

**`<IonIcon>` vs `<ion-icon>` web component**: In React projects, always use `<IonIcon>` — it provides TypeScript types, React event handling, and integrates with Ionic's component system. The raw web component works but loses type safety and React integration.

**Named import vs CDN**: Named imports from `ionicons/icons` enable tree-shaking and work offline. CDN scripts are simpler but load the entire icon set and require a network. Always use named imports in production builds.

**Filled vs Outline variant**: Filled icons carry more visual weight — use for active/selected state. Outline icons are lighter — use for inactive state or default. Be consistent: don't mix conventions within the same UI pattern.

---

## Implementation Notes

- Ionicons 7 is included automatically with `@ionic/react` — no separate install needed.
- For standalone use: `pnpm add ionicons`.
- Browser support: Chrome 79+, Safari 14+, Edge 79+, Firefox 70+. No IE 11 support.
- Stroke width of outline icons is controlled by `--ionicon-stroke-width` CSS custom property (default: `32px`).
- Icon names use camelCase in JS imports (`heartOutline`) and kebab-case as web component names (`heart-outline`).
- Use the Ionicons search tool at https://ionic.io/ionicons to find icon names before importing.
