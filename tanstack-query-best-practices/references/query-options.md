# queryOptions Usage

Define a `queryOptions` factory for each reusable query. Reuse it in components,
loaders, mutations, and cache operations instead of rebuilding query keys or
query functions.

```ts
import { queryOptions } from "@tanstack/react-query";

import { getUsers } from "./api";
import type { UserFilters } from "../user-filter";

export const usersQueryOptions = (filters: UserFilters) =>
	queryOptions({
		queryKey: ["users", filters],
		queryFn: () => getUsers(filters),
	});
```

## Query Factory Object

Use a query factory object when a feature owns multiple related queries. Keep
prefix keys and complete query options together while preserving `queryOptions`
type inference.

```ts
import { queryOptions } from "@tanstack/react-query";

import { getUser, getUsers } from "./api";
import type { UserFilters, UserId } from "../user-filter";

export const userQueries = {
	all: () => {
		return ["users"];
	},
	lists: () => {
		return [...userQueries.all(), "list"];
	},
	list: (filters: UserFilters) => {
		return queryOptions({
			queryKey: [...userQueries.lists(), filters],
			queryFn: () => getUsers(filters),
		});
	},
	details: () => {
		return [...userQueries.all(), "detail"];
	},
	detail: (userId: UserId) => {
		return queryOptions({
			queryKey: [...userQueries.details(), userId],
			queryFn: () => getUser(userId),
		});
	},
};
```

- Use prefix helpers such as `all()`, `lists()`, and `details()` for broad cache
  matching.
- Use complete options such as `list(filters)` and `detail(userId)` everywhere
  else. Read `.queryKey` only when an API requires a key.
- Expose one public factory API instead of duplicate standalone keys and option
  factories.

```ts
const usersQuery = useSuspenseQuery(userQueries.list(filters));
await queryClient.invalidateQueries({ queryKey: userQueries.all() });
const user = queryClient.getQueryData(userQueries.detail(userId).queryKey);
```

## Consumers

Use the same factory with regular and Suspense consumers.

```ts
const usersQuery = useQuery(usersQueryOptions(filters));
const suspenseUsersQuery = useSuspenseQuery(usersQueryOptions(filters));
```

Use `useSuspenseQueries` with `combine` to read and merge several queries under
one Suspense boundary. The results are settled, so `combine` can read `data`
without loading guards.

```ts
const { users, posts } = useSuspenseQueries({
	queries: [usersQueryOptions(filters), postsQueryOptions()],
	combine: ([usersResult, postsResult]) => ({
		users: usersResult.data.users,
		posts: postsResult.data.posts,
	}),
});
```

Keep `combine` pure and return stable derived values. It can also fold a dynamic
queries array into one result.

## Query Client Operations

Pass complete options when the API accepts them. Use `.queryKey` only for
key-only APIs such as `getQueryData` and `setQueryData`.

```ts
await queryClient.invalidateQueries(usersQueryOptions(filters));
await queryClient.refetchQueries(usersQueryOptions(filters));
await queryClient.cancelQueries(usersQueryOptions(filters));

const data = queryClient.getQueryData(usersQueryOptions(filters).queryKey);

queryClient.setQueryData(usersQueryOptions(filters).queryKey, (previous) => {
	if (!previous) return previous;

	return {
		...previous,
		activeCount: previous.items.filter((user) => user.status === "active").length,
	};
});

await queryClient.ensureQueryData(usersQueryOptions(filters));
queryClient.prefetchQuery(usersQueryOptions(filters));
```
