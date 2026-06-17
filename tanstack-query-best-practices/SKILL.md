---
name: tanstack-query-best-practices
description: Apply user-defined TanStack Query best practices for React apps. Use when implementing, refactoring, or reviewing @tanstack/react-query usage, especially queryOptions, selectors, mutations, and integration with TanStack Router.
---

# TanStack Query Best Practices

Load only the reference file that matches the task:

- `references/query-options.md` for `queryOptions`, query keys, query functions, query consumers, and query client operations.
- `references/selectors.md` for `select`, selected subscriptions, selector function placement, and derived query slices.
- `references/mutations.md` for `useMutation`, mutation options, invalidation, optimistic updates, and mutation side effects.
- `references/mutation-state.md` for `mutationOptions`, `mutationKey`, `useMutationState`, `useIsMutating`, and observing mutation state outside the mutation owner.
- `references/router.md` for TanStack Query usage with TanStack Router routes, loaders, search params, and preloading.
- `references/dependent-queries.md` for `enabled`, conditional queries, dependent queries, required query inputs, and modal/drawer-gated fetching.
- `references/suspense-error-handling.md` for Suspense query errors, route error boundaries, query reset behavior, retries, and `router.invalidate()`.
- `references/infinite-queries.md` for `infiniteQueryOptions`, `useSuspenseInfiniteQuery`, page params, pagination cursors, and infinite query invalidation.
