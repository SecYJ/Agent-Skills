# Select Usage

Use `select` when a consumer needs only part of a query result or a value derived from it. `select` does not change the cached data.

Selector shape depends on whether React Compiler applies to the affected code. Check that package's compiler config if unknown; if inconclusive, follow nearby selectors.

## With React Compiler

Inline the selector and leave `data` unannotated so it is inferred. No `useCallback` or module hoisting just for stability.

```ts
const activeUserCountQuery = useSuspenseQuery({
	...usersQueryOptions(filters),
	select(data) {
		return data.activeCount;
	},
});
```

## Without React Compiler

Wrap the inline selector in `useCallback` so it does not rerun on unrelated renders. Annotate `data` and list captured values as dependencies.

```ts
select: useCallback((data: UsersResult) => data.activeCount, []),
```

## useQueries / useSuspenseQueries

These do not infer the per-query `select` parameter. Always annotate it, with or without the compiler.

```ts
const [activeUserCountQuery] = useSuspenseQueries({
	queries: [
		{
			...usersQueryOptions(filters),
			select(data: UsersResult) {
				return data.activeCount;
			},
		},
	],
});
```

## Shared Selectors

Hoist a selector to module scope when several consumers share it.

```ts
function selectActiveUserNames(data: UsersResult) {
	return data.items.filter((user) => user.status === "active").map((user) => user.name);
}
```

When a consumer needs several values from one query, return one object from a single `select` rather than calling the query multiple times.
