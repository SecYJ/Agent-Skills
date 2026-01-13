---
name: shadcn-theme-polish
description: Create beautiful, consistent, accessible UIs with shadcn (Base UI) + Tailwind CSS v4, with strong light/dark theming. Use when asked to polish layout/typography/spacing, standardize styling tokens, add or fix dark mode, or build new pages/components that match the existing design system.
---

# Shadcn Theme Polish

## Overview

Turn “working UI” into “polished product UI” by enforcing a consistent design system (tokens, spacing, typography, surfaces, interaction states) and a robust light/dark theme.

This repo already uses semantic shadcn-style tokens via `src/styles/app.css` (e.g. `bg-background`, `text-foreground`, `border-border`, `ring-ring`) and a `.dark` class for dark mode. Prefer these tokens over ad-hoc colors.

## Workflow

1. Clarify intent and constraints
   - What screen is this (dashboard, list+filters, settings, form)?
   - What must remain (data, copy, interactions)?
   - Target: “clean + modern”, “compact + enterprise”, or “friendly + spacious”.

2. Convert styling to semantic tokens (don’t fight the theme)
   - Replace raw colors like `bg-white`, `text-black`, `bg-slate-200`, `dark:bg-slate-800` with:
     - Surfaces: `bg-background`, `bg-card`, `bg-popover`
     - Text: `text-foreground`, `text-muted-foreground`
     - Borders: `border-border`
     - Focus: `ring-ring` + `ring-offset-background`
   - Use `bg-primary text-primary-foreground` for primary actions.

3. Apply layout and typography polish
   - Use a stable page grid: a centered container with consistent padding and section rhythm.
   - Establish hierarchy: 1 strong headline, 1 subhead, then content.
   - Ensure responsive behavior at common breakpoints.

4. Make interactions feel “real”
   - Hover, active, disabled, loading, focus-visible states.
   - Consistent button sizes and icon alignment.
   - Empty states and error states.

5. Validate light + dark + accessibility
   - Check contrast (especially muted text), focus rings, and keyboard navigation.
   - Ensure both themes look intentional (not “inverted”).

## Project Rules (do these by default)

- Prefer shadcn/Base UI patterns and tokens over bespoke styling.
- Avoid introducing `@radix-ui/*` dependencies unless explicitly requested.
- Prefer using the shadcn MCP server (if available) to generate components; verify output is Base UI-flavored.
- Keep changes surgical: don’t re-theme the whole app unless asked.

## Recipes

### Page container + rhythm

Start pages with a consistent structure:

```
<main className="min-h-dvh bg-background text-foreground">
  <div className="mx-auto w-full max-w-6xl px-4 py-10 md:px-6">
    ...
  </div>
</main>
```

Use vertical rhythm with `space-y-*` / `gap-*` rather than scattered margins.

### Typography baseline

- Headline: `text-3xl md:text-4xl font-semibold tracking-tight`
- Subhead: `text-base text-muted-foreground`
- Section title: `text-lg font-semibold`
- Supporting: `text-sm text-muted-foreground`

### Buttons and focus states

Prefer semantic styles over raw colors:
- Primary: `bg-primary text-primary-foreground`
- Subtle/secondary: `bg-secondary text-secondary-foreground`
- Destructive: `bg-destructive text-primary-foreground` (or component default)

For custom interactive elements, ensure focus-visible rings (example pattern):

```
focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 ring-offset-background
```

## Light/Dark Theme Checklist

- Theme tokens live in `src/styles/app.css`. Prefer `bg-*`/`text-*`/`border-*` token classes that map to those variables.
- Dark mode is driven by the `.dark` class (apply it to `<html>` or a top-level wrapper).
- Avoid mixing token colors with hardcoded grays on the same surface (it usually looks “off” in one theme).
- Optional but recommended: set `color-scheme` to match theme in CSS/JS when implementing a theme toggle.

## References

- Theme toggle implementation: `references/theme-toggle.md`
- Layout patterns: `references/layout-recipes.md`
- QA checklist: `references/polish-checklist.md`
