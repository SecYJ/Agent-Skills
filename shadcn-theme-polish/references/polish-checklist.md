# UI Polish Checklist

Use this to review any screen after changes.

## Theme + color

- Uses semantic tokens (`bg-background`, `bg-card`, `text-foreground`, `border-border`).
- Primary/secondary actions are visually distinct.
- Muted text is still readable in both themes.

## Typography

- Clear hierarchy (H1 → section titles → body).
- Line length is reasonable (`max-w-prose` for long text).
- Code blocks / monospace are used sparingly and consistently.

## Layout + spacing

- Consistent container width and padding across pages.
- Vertical rhythm is consistent (use `space-y-*` / `gap-*`).
- Cards align; no “almost aligned” edges.

## Interaction states

- Focus-visible ring is present and consistent.
- Disabled states are obvious.
- Loading states don’t cause layout jump.

## Accessibility

- Everything interactive is keyboard reachable.
- Tap targets are at least ~40px tall on mobile.
- Color is not the only indicator (errors, selection, etc.).
