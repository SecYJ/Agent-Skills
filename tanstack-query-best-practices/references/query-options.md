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
