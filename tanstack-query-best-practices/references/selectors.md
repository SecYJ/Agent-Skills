# Select Usage

Use `select` when a consumer only needs part of a query result or a value
derived from it.

## Choose Selector Shape

Check the project configuration to determine whether React Compiler is enabled.

### With React Compiler

Inline the selector. Do not add `useCallback` or move it to module scope only
for reference stability. For a single query with spread query options, leave the
`data` parameter unannotated so its type is inferred.

```ts
const activeUserCountQuery = useSuspenseQuery({
	...usersQueryOptions(filters),
	select: (data) => data.activeCount,
});
```

`useQueries` and `useSuspenseQueries` do not infer the per-query `select`
parameter from the queries array. Annotate it explicitly, even with React
Compiler enabled.

```ts
const [activeUserCountQuery] = useSuspenseQueries({
	queries: [
		{
			...usersQueryOptions(filters),
			select: (data: UsersResult) => data.activeCount,
		},
	],
});
```

### Without React Compiler

Wrap an inline selector in `useCallback` and annotate its `data` parameter.
Include component values captured by the selector in the dependency array.

```ts
const activeUserCountQuery = useSuspenseQuery({
	...usersQueryOptions(filters),
	select: useCallback((data: UsersResult) => data.activeCount, []),
});
```

### Shared Selector

Move a selector to module scope when multiple consumers share it or doing so
improves clarity.

```ts
const selectActiveUserCount = (data: UsersResult) => data.activeCount;

const activeUserCountQuery = useSuspenseQuery({
	...usersQueryOptions(filters),
	select: selectActiveUserCount,
});
```

### Multiple Selected Values

Return one selected object when a consumer needs several values from the same
query.

```ts
const { totalCount, activeCount } = useSuspenseQuery({
	...usersQueryOptions(filters),
	select: (data) => ({
		totalCount: data.totalCount,
		activeCount: data.activeCount,
	}),
}).data;
```

### Derived Values

Use `select` to derive consumer-specific values without changing the cached
query data.

```ts
const { data: activeUserNames } = useSuspenseQuery({
	...usersQueryOptions(filters),
	select(data) {
		return data.items
			.filter((user) => user.status === "active")
			.map((user) => user.name);
	},
});
```
