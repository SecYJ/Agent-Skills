# Dependent Query Usage

Use `enabled` when a query must wait for a required input. Include that input in
the query key and guard the query function so its type remains safe.

```ts
export function userQueryOptions(userId?: string) {
	return queryOptions({
		queryKey: ["users", userId],
		queryFn() {
			if (!userId) throw new Error("userId is required");

			return getUser(userId);
		},
	});
}

const userQuery = useQuery({
	...userQueryOptions(userId),
	enabled: Boolean(userId),
});
```

Use `useQuery` for queries that can be disabled. Reserve `useSuspenseQuery` for
queries whose required inputs are already available.

## Dependent On Another Query

Pass the value produced by the first query into the dependent query options.

```ts
const currentUserQuery = useSuspenseQuery(currentUserQueryOptions());
const managerId = currentUserQuery.data.managerId;

const managerQuery = useQuery({
	...userQueryOptions(managerId),
	enabled: Boolean(managerId),
});
```

## Modal And Drawer Queries

Include visibility in `enabled` when a query should run only while its UI is
open.

```ts
const userPreviewQuery = useQuery({
	...userQueryOptions(selectedUserId),
	enabled: open && Boolean(selectedUserId),
});
```
