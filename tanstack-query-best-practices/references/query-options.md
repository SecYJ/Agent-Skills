# queryOptions Usage

Define a `queryOptions` factory for each reusable query.

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

The factory becomes the single source of truth for the query because the `queryKey`, `queryFn`, and default query behavior live in one place. Components, loaders, mutations, and cache operations all reuse the same options instead of rebuilding matching keys by hand.

## Query Factory Object

Use a query factory object when a feature owns multiple related queries. The object is the single source of truth for the feature's query keys and `queryOptions`, following the query-factory structure while keeping TanStack Query's `queryOptions` type inference.

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

Keep key-only helpers and full query option helpers on the same object:

- Use prefix key helpers such as `userQueries.all()`, `userQueries.lists()`, and `userQueries.details()` for broad cache matching.
- Use `queryOptions` helpers such as `userQueries.list(filters)` and `userQueries.detail(userId)` for reads, prefetching, cache seeding, invalidation, refetching, and cancellation.
- Use `.queryKey` from the `queryOptions` helper only when an API needs a key instead of a full options object, such as `getQueryData` or `setQueryData`.
- Do not export duplicate standalone keys or compatibility `usersQueryOptions` functions once the object factory is the public API.

```ts
const usersQuery = useSuspenseQuery(userQueries.list(filters));
```

```ts
await queryClient.invalidateQueries({
	queryKey: userQueries.all(),
});
```

```ts
const user = queryClient.getQueryData(userQueries.detail(userId).queryKey);
```

## Consumers

Use the factory anywhere the query is read.

```ts
const usersQuery = useQuery(usersQueryOptions(filters));
```

```ts
const usersQuery = useSuspenseQuery(usersQueryOptions(filters));
```

## Query Client Operations

Reuse the same factory for imperative query client operations.

```ts
await queryClient.invalidateQueries(usersQueryOptions(filters));
```

```ts
await queryClient.refetchQueries(usersQueryOptions(filters));
```

```ts
await queryClient.cancelQueries(usersQueryOptions(filters));
```

```ts
const data = queryClient.getQueryData(usersQueryOptions(filters).queryKey);
```

```ts
queryClient.setQueryData(usersQueryOptions(filters).queryKey, (previous) => {
	if (!previous) return previous;

	return {
		...previous,
		activeCount: previous.items.filter((user) => user.status === "active").length,
	};
});
```

```ts
await queryClient.ensureQueryData(usersQueryOptions(filters));
```

```ts
queryClient.prefetchQuery(usersQueryOptions(filters));
```
