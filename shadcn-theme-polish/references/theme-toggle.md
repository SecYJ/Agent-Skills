# Theme Toggle (Light / Dark / System)

Goal: toggle the `.dark` class (used by Tailwind v4 `@custom-variant dark`) and persist user preference.

## Minimal approach (recommended)

1. Use a `theme` preference: `"light" | "dark" | "system"`.
2. Persist to `localStorage` (e.g. key `theme`).
3. Compute the effective theme:
   - If `"system"`, use `window.matchMedia('(prefers-color-scheme: dark)')`.
   - Otherwise use the stored value.
4. Apply to `<html>`:
   - `document.documentElement.classList.toggle('dark', effectiveTheme === 'dark')`

## UX details

- Default to `"system"` unless the product wants “always light”.
- Apply early to avoid a flash (hydrate-safe approach may require an inline script or server-rendered class).
- Optional: update `document.documentElement.style.colorScheme = effectiveTheme`.

## Styling rule

Once theme is wired, replace ad-hoc theme classes like `dark:bg-slate-800` with semantic tokens:
- `bg-background`, `bg-card`, `text-foreground`, `text-muted-foreground`, `border-border`.
