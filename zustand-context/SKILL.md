---
name: zustand-context
description: Scaffold a scoped Zustand store with React Context provider. Use when creating a new feature context/provider, setting up scoped state management, or when the user asks to create a provider with Zustand. Trigger on phrases like "scoped context", "zustand context", "scoped store".
argument-hint: <FeatureName> [output-path]
---

# Zustand Scoped Context Provider

Generate a scoped Zustand store + React Context provider following the project's established pattern.

## Input

- `$ARGUMENTS[0]` — **Feature name** in PascalCase (e.g. `Checkout`, `Profile`, `TransactionHistory`)
- `$ARGUMENTS[1]` — (optional) Output file path. If omitted, ask the user where to place the file.

If the user only gives a feature name without specifying state fields, generate a minimal scaffold with one example state field (`isLoading: boolean`) and a corresponding action (`setIsLoading`) so the structure is clear and ready to be filled in. Add a `TODO` comment above the state type prompting the user to replace with real fields.

## Template

Generate a single `.tsx` file with this exact structure. Every section matters — do not skip or reorder any part.

```tsx
import { createContext, type ReactNode, use, useState } from "react"
import { createStore, useStore } from "zustand"

// TODO: Replace with actual state fields and actions for this feature
type {{Name}}State = {
  // ... state fields here
  actions: {
    // ... action methods here
  }
}

type {{Name}}Store = ReturnType<typeof create{{Name}}Store>

type {{Name}}ContextValue = {
  store: {{Name}}Store
}

type {{Name}}ProviderProps = {
  children: ReactNode
  // Add optional initialX props here if the provider needs initial values
}

const create{{Name}}Store = (/* accept initial value params here if needed */) => {
  return createStore<{{Name}}State>()((set) => ({
    // ... initial state values
    actions: {
      // ... action implementations using set()
    },
  }))
}

const {{Name}}Context = createContext<{{Name}}Store | null>(null)

export const {{Name}}Provider = ({ children }: {{Name}}ProviderProps) => {
  const [store] = useState(create{{Name}}Store)

  return <{{Name}}Context value={store}>{children}</{{Name}}Context>
}

export const use{{Name}} = <T,>(selector: (state: {{Name}}State) => T) => {
  const store = use({{Name}}Context)

  if (!context) throw new Error("{{Name}}Provider is being used outside of {{Name}}Context")

  return useStore(store, selector)
}
```

## Conventions to follow exactly

- **Actions inside state**: all mutators live under an `actions` object so consumers destructure cleanly: `const { setX } = useFeature(s => s.actions)`
- **`use()` from React 19**: use `use(Context)`, NOT `useContext(Context)`
- **JSX `value=` prop**: write `<Context value={{ store }}>`, NOT `<Context.Provider value={{ store }}>`
- **`useState` for store creation**: `const [store] = useState(createStore)` — never `useRef`
- **Selector-only hook**: the public hook always requires a selector `(state: State) => T` for render optimization
- **Null context default**: `createContext<ContextValue | null>(null)` with a runtime error guard
- **Named exports only**: `export const`, no default exports
- **No barrel files**: do not create an `index.ts` unless asked
- **Store type alias**: always `type XStore = ReturnType<typeof createXStore>`

## When the user provides state fields

If the user describes specific state (e.g. "I need `currentStep`, `formData`, and `errors`"), wire them into the type and store factory with matching actions. Use your judgment for action names — follow the `setX` convention for simple setters. For complex state transitions, name the action after what it does (e.g. `advanceStep`, `resetForm`).

## When initial values are needed

If the provider needs to accept initial values as props (like `LoginProvider` does with `initialView`):

1. Add the initial value props to `{{Name}}ProviderProps` (with `?` optional marker and defaults)
2. Accept them in `create{{Name}}Store` via an `Omit<{{Name}}ProviderProps, "children">` parameter
3. Pass them through in the `useState` initializer

Only do this if the user asks for it or it's clearly needed. The simpler pattern (no initial props, like `RegisterProvider`) is the default.
