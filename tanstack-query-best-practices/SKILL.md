---
name: tanstack-query-best-practices
description: Use only when writing or editing @tanstack/react-query code, including queries, mutations, cache operations, and TanStack Router loader integration. NEVER use this skill for reading, explaining, or reviewing code, or for React or Router work that does not change TanStack Query code.
---

# TanStack Query Best Practices

Load only the references the task needs; add another when the work crosses topics. Follow nearby code for routine choices.

Examples use `function` declarations for named functions and method shorthand for function-valued object properties.

- `references/query-options.md`: `queryOptions`, query keys, query factories, consumers, `useSuspenseQueries`, query client operations.
- `references/selectors.md`: `select`, selector placement, derived query slices.
- `references/mutations.md`: `useMutation`, invalidation, optimistic updates, mutation callbacks.
- `references/mutation-state.md`: `mutationOptions`, `mutationKey`, `useMutationState`, `useIsMutating`.
- `references/router.md`: TanStack Router loaders, `loaderDeps`, query options in route context, Suspense consumers.
- `references/dependent-queries.md`: `skipToken`, `enabled`, conditional and dependent queries, modal/drawer-gated fetching.
- `references/suspense-error-handling.md`: Suspense errors, error boundaries, query reset, retries, `router.invalidate()`.
- `references/infinite-queries.md`: `infiniteQueryOptions`, page params, cursors, infinite query invalidation.
