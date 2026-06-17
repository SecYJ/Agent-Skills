# Select Usage

Use `select` when a consumer only needs part of a query result.

```ts
const selectActiveUserCount = (data: UsersResult) => data.activeCount;

const activeUserCountQuery = useSuspenseQuery({
	...usersQueryOptions(filters),
	select: selectActiveUserCount,
});
```

Keep selector functions stable at module scope. Prefer a named selector
reference over inline selector callbacks.

When a consumer needs multiple values from the same query, return one selected object and destructure it.

```ts
const selectUserSummary = (data: UsersResult) => ({
	totalCount: data.totalCount,
	activeCount: data.activeCount,
});

const { totalCount, activeCount } = useSuspenseQuery({
	...usersQueryOptions(filters),
	select: selectUserSummary,
}).data;
```
