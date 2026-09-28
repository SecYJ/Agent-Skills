# queryOptions Usage

Define a `queryOptions` factory for each reusable query and reuse it in components, loaders, mutations, and cache operations instead of rebuilding keys or query functions.

Examples assume `getUsers` returns `{ items: User[]; totalCount: number; activeCount: number }` (`UsersResult`). Match the real API shape in application code.

```ts
import { queryOptions } from "@tanstack/react-query";

export function usersQueryOptions(filters: UserFilters) {
	return queryOptions({
		queryKey: ["users", filters],
		queryFn() {
			return getUsers(filters);
		},
	});
}
```

## Query Factory Object

When a feature owns several related queries, group prefix keys and complete options in one object.

```ts
export const userQueries = {
	all() {
		return ["users"];
	},
	lists() {
		return [...userQueries.all(), "list"];
	},
	list(filters: UserFilters) {
		return queryOptions({
			queryKey: [...userQueries.lists(), filters],
			queryFn() {
				return getUsers(filters);
			},
		});
	},
	details() {
		return [...userQueries.all(), "detail"];
	},
	detail(userId: UserId) {
		return queryOptions({
			queryKey: [...userQueries.details(), userId],
			queryFn() {
				return getUser(userId);
			},
		});
	},
};
```

- Use prefix helpers (`all()`, `lists()`, `details()`) only for broad cache matching.
- Use complete options (`list(filters)`, `detail(userId)`) everywhere else.
- Expose this one API; do not also export standalone keys or duplicate factories.

```ts
const usersQuery = useSuspenseQuery(userQueries.list(filters));
await queryClient.invalidateQueries({ queryKey: userQueries.all() });
```

## Multiple Suspense Queries

When one component needs more than one Suspense query, replace the separate `useSuspenseQuery` calls with one `useSuspenseQueries`. Separate calls suspend one after another and fetch in a waterfall; `useSuspenseQueries` fetches in parallel.

Use `combine` to shape the result. Results are settled, so no loading guards are needed. Keep `combine` pure.

```ts
const { users, posts } = useSuspenseQueries({
	queries: [usersQueryOptions(filters), postsQueryOptions()],
	combine([usersResult, postsResult]) {
		return {
			users: usersResult.data.items,
			posts: postsResult.data.posts,
		};
	},
});
```

## Query Client Operations

Pass complete options wherever the API accepts them. Use `.queryKey` only for key-only APIs such as `getQueryData` and `setQueryData`.

```ts
await queryClient.invalidateQueries(usersQueryOptions(filters));

queryClient.setQueryData(usersQueryOptions(filters).queryKey, (previous) => {
	if (!previous) return previous;

	return {
		...previous,
		activeCount: previous.items.filter((user) => user.status === "active").length,
	};
});
```

`ensureQueryData`, `refetchQueries`, `cancelQueries`, and `prefetchQuery` accept options the same way.
