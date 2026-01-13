# Layout Recipes (Tailwind + shadcn tokens)

Use these as starting points, then iterate.

## Marketing-ish landing / welcome

- Outer: `min-h-dvh bg-background text-foreground`
- Container: `mx-auto max-w-6xl px-4 py-12 md:px-6`
- Hero: `space-y-3`
- Hero title: `text-4xl md:text-5xl font-semibold tracking-tight`
- Hero subtitle: `text-base md:text-lg text-muted-foreground max-w-prose`
- Card grid: `mt-10 grid gap-4 sm:grid-cols-2 lg:grid-cols-3`
- Card: `rounded-xl border border-border bg-card text-card-foreground p-5`

## App page (header + content)

- Page: `min-h-dvh bg-background text-foreground`
- Header row: `flex items-start justify-between gap-4`
- Title block: `space-y-1`
- Actions: `flex flex-wrap items-center gap-2`
- Content: `mt-8 grid gap-6 lg:grid-cols-[280px_1fr]`
- Sidebar: `rounded-xl border border-border bg-card p-4`
- Main: `rounded-xl border border-border bg-card p-6`

## List + filters

- Filters row: `flex flex-col gap-3 md:flex-row md:items-end md:justify-between`
- Search input group: `flex-1`
- Table/list wrapper: `mt-6 rounded-xl border border-border bg-card`
- Empty state: center copy + primary CTA (keep it short)

## Forms (settings-style)

- Stack: `space-y-8`
- Section card: `rounded-xl border border-border bg-card p-6`
- Section header: `mb-4 space-y-1`
- Label: `text-sm font-medium`
- Help text: `text-sm text-muted-foreground`
