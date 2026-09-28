---
name: zustand-context
description: Scaffold a scoped Zustand store exposed through a React Context provider. Use when creating a feature-scoped store, provider, or "zustand context".
argument-hint: <FeatureName> [output-path]
---

# Zustand Scoped Context Provider

Generate one `.tsx` file containing a per-instance Zustand store and its provider.

## Input

- `$ARGUMENTS[0]`: feature name in PascalCase, e.g. `Checkout`.
- `$ARGUMENTS[1]`: optional output path. If omitted, place it beside the feature following nearby conventions; ask only if there is no clear location.

If no state fields are given, scaffold one example field `isLoading: boolean` with a `setIsLoading` action and a `TODO` above the state type.

## Template

```tsx
import { createContext, type ReactNode, use, useState } from "react"
import { createStore, useStore } from "zustand"

// TODO: Replace with this feature's state and actions
type {{Name}}State = {
  isLoading: boolean
  actions: {
    setIsLoading: (isLoading: boolean) => void
  }
}

type {{Name}}Store = ReturnType<typeof create{{Name}}Store>

type {{Name}}ProviderProps = {
  children: ReactNode
}

function create{{Name}}Store() {
  return createStore<{{Name}}State>()((set) => ({
    isLoading: false,
    actions: {
      setIsLoading(isLoading) {
        set({ isLoading })
      },
    },
  }))
}

const {{Name}}Context = createContext<{{Name}}Store | null>(null)

export function {{Name}}Provider({ children }: {{Name}}ProviderProps) {
  const [store] = useState(create{{Name}}Store)

  return <{{Name}}Context value={store}>{children}</{{Name}}Context>
}

export function use{{Name}}<T>(selector: (state: {{Name}}State) => T) {
  const store = use({{Name}}Context)

  if (!store) throw new Error("use{{Name}} must be used within {{Name}}Provider")

  return useStore(store, selector)
}
```

## Conventions

- Use `function` declarations for the factory, provider, and hook, and method shorthand for actions.
- Put all mutators under `actions` so consumers read them in one stable selection: `const { setX } = useX((s) => s.actions)`.
- React 19 APIs: `use(Context)` and `<Context value={...}>`, not `useContext` or `<Context.Provider>`.
- Create the store with `useState(factory)`, never `useRef`.
- The public hook always takes a selector. For selectors returning new objects or arrays, use `useShallow`.
- Named exports only; no barrel `index.ts` unless asked.

## Custom State

Wire requested fields into the state type and factory. Use `setX` for plain setters and intent names (`advanceStep`, `resetForm`) for real transitions.

## Initial Values

Only when the provider needs initial values as props:

```tsx
type {{Name}}ProviderProps = {
  children: ReactNode
  initialStep?: number
}

function create{{Name}}Store({ initialStep = 0 }: Omit<{{Name}}ProviderProps, "children">) {
  return createStore<{{Name}}State>()((set) => ({ step: initialStep, /* ... */ }))
}

export function {{Name}}Provider({ children, ...initial }: {{Name}}ProviderProps) {
  const [store] = useState(() => create{{Name}}Store(initial))
  // ...
}
```

Initial props are read once; later prop changes do not update the store.
