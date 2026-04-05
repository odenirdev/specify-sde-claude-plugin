# Capacitor — Engineering Reference

## Best Practices

### Plugin Usage
- Always check official Capacitor plugins first before using community or Cordova plugins.
- Install official plugins from `@capacitor/` scope: `pnpm add @capacitor/camera @capacitor/filesystem`.
- Await `addListener` — in Capacitor 6, it returns a `Promise<PluginListenerHandle>`, not a sync value:
  ```ts
  const handle = await App.addListener('appStateChange', ({ isActive }) => { ... });
  // cleanup:
  await handle.remove();
  ```
- Always remove listeners on component unmount to prevent memory leaks.

### Platform Detection
- Use `Capacitor.getPlatform()` for platform branching — never `navigator.userAgent`:
  ```ts
  import { Capacitor } from '@capacitor/core';
  const platform = Capacitor.getPlatform(); // 'ios' | 'android' | 'web'
  ```
- Use `Capacitor.isNativePlatform()` to distinguish hybrid from web:
  ```ts
  if (Capacitor.isNativePlatform()) { /* native only logic */ }
  ```
- Use `Capacitor.isPluginAvailable('Camera')` before calling a plugin to handle web gracefully.

### Permissions
- Always check and request permissions before calling restricted APIs:
  ```ts
  const perms = await Camera.checkPermissions();
  if (perms.camera !== 'granted') {
    await Camera.requestPermissions({ permissions: ['camera'] });
  }
  ```
- Handle `denied` state explicitly — do not silently fail after a refused permission.

### Configuration (capacitor.config.ts)
- Use `capacitor.config.ts` (TypeScript) for type-safe config:
  ```ts
  import { CapacitorConfig } from '@capacitor/cli';
  const config: CapacitorConfig = {
    appId: 'com.example.app',
    appName: 'MyApp',
    webDir: 'dist',
    android: { allowMixedContent: false },
    ios: { contentInset: 'always' },
  };
  export default config;
  ```
- Set `androidScheme: 'https'` (default in Cap 6) — do not downgrade to `http` without understanding cookie/localStorage implications.
- Keep `webDir` aligned with your build tool's output directory.

### Native Builds
- Run `npx cap sync` after installing a Capacitor plugin — it copies web assets and native plugin code.
- Run `npx cap copy` when only web assets changed (faster than sync).
- Never edit auto-generated native code inside `android/app/src/main/assets/public` or `ios/App/public` — it is overwritten on every sync.
- Use `npx cap migrate` when upgrading Capacitor major versions.

### iOS Specifics (Capacitor 6)
- Custom local plugins must be registered explicitly in a custom `AppDelegate` — they are no longer auto-discovered:
  ```swift
  // AppDelegate.swift
  override func application(...) {
    self.bridge?.registerPluginInstance(MyPlugin())
  }
  ```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `navigator.userAgent` for platform detection | Fragile, breaks in WebView | Use `Capacitor.getPlatform()` |
| Not awaiting `addListener` (Cap 6) | Silent failure, returns Promise not handle | Always `await` addListener |
| Not removing listeners on unmount | Memory leaks, ghost callbacks | Store handle, call `handle.remove()` |
| Calling restricted API without permission check | Runtime crash on iOS/Android | Check permissions first |
| Editing files inside `android/*/assets/public` | Overwritten on next `cap sync` | Edit source, then sync |
| Using `androidScheme: 'http'` in new apps | Breaks autofill, security downgrade | Keep default `https` |
| Relying on Cordova plugin compatibility long-term | Unmaintained, not Capacitor-native | Migrate to Capacitor plugin |

---

## Review Criteria

- [ ] `Capacitor.getPlatform()` used for all platform checks — no user-agent sniffing
- [ ] `addListener` is `await`ed and result stored for later cleanup
- [ ] Listeners removed in component unmount / effect cleanup
- [ ] Permissions checked before calling restricted native APIs
- [ ] `capacitor.config.ts` uses typed `CapacitorConfig`
- [ ] `npx cap sync` runs in CI after plugin install
- [ ] No manual edits inside native `public/` asset directories

---

## Trade-offs

**Capacitor vs Cordova**: Capacitor is web-first with native opt-in and explicit plugin registration. Cordova requires more native boilerplate. Choose Capacitor for all new Ionic projects — Cordova is in maintenance mode.

**Official plugins vs community plugins**: Official `@capacitor/` plugins are maintained by the Ionic team, tested against each major version, and ship TypeScript types. Community plugins may lag major releases. Prefer official; audit community plugins for last update date before using.

**`capacitor.config.ts` vs `capacitor.config.json`**: TypeScript config provides autocompletion and type checking. Use `.ts` for all projects with TypeScript.

---

## Implementation Notes

- Install: `pnpm add @capacitor/core @capacitor/cli @capacitor/ios @capacitor/android`.
- Init: `npx cap init [AppName] [AppId] --web-dir dist`.
- Add platforms: `npx cap add ios && npx cap add android`.
- Capacitor 6 requires **Node 18+**, **Xcode 15+**, **Android Studio Hedgehog (2023.1.1+)**.
- Gradle 8.2 is required for Android builds in Capacitor 6.
- Run `npx cap open ios` or `npx cap open android` to open native IDEs.
- Test web fallbacks with `Capacitor.isPluginAvailable()` — not all plugins work in browser.
