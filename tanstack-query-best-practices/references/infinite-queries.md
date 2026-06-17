# Infinite Query Usage

Use `infiniteQueryOptions` for cursor or page based lists that load more data
from the same query family.

```ts
import { infiniteQueryOptions, useInfiniteQuery, useSuspenseInfiniteQuery } from "@tanstack/react-query";

import { getUsers } from "./api";
import type { UserFilters } from "../user-filter";

export const usersInfiniteQueryOptions = (filters: UserFilters) => {
	return infiniteQueryOptions({
		queryKey: ["users", "infinite", filters],
		queryFn: ({ pageParam }) =>
			getUsers({
				...filters,
				cursor: pageParam,
			}),
		initialPageParam: undefined as string | undefined,
		getNextPageParam: (lastPage) => lastPage.nextCursor,
	});
};
```

The query key must include every stable input that changes the list identity,
such as filters, search text, sort order, and page size. Do not include
`pageParam` in the query key because TanStack Query manages pages inside the
infinite query cache entry.

## Consumers

Use `useSuspenseInfiniteQuery` for route page data.

```tsx
const UsersPage = ({ filters }: { filters: UserFilters }) => {
	const usersQuery = useSuspenseInfiniteQuery(usersInfiniteQueryOptions(filters));
	const users = usersQuery.data.pages.flatMap((page) => page.items);

	return (
		<UsersList
			users={users}
			fetchNextPage={usersQuery.fetchNextPage}
			hasNextPage={usersQuery.hasNextPage}
			isFetchingNextPage={usersQuery.isFetchingNextPage}
		/>
	);
};
```

Use `useInfiniteQuery` when the infinite query is disabled or should not
suspend.

```tsx
const UserPicker = ({ filters, open }: { filters: UserFilters; open: boolean }) => {
	const usersQuery = useInfiniteQuery({
		...usersInfiniteQueryOptions(filters),
		enabled: open,
	});
	const users = usersQuery.data?.pages.flatMap((page) => page.items) ?? [];

	return (
		<UsersList
			users={users}
			fetchNextPage={usersQuery.fetchNextPage}
			hasNextPage={usersQuery.hasNextPage}
			isFetchingNextPage={usersQuery.isFetchingNextPage}
		/>
	);
};
```

## Select Flattened Data

Use a stable `select` function when multiple consumers need the same flattened
shape. Keep the full `pages` and `pageParams` structure so infinite query
metadata remains intact.

```ts
type UsersPageResult = Awaited<ReturnType<typeof getUsers>>;

const selectFlattenedUsers = (data: { pages: UsersPageResult[]; pageParams: unknown[] }) => ({
	...data,
	users: data.pages.flatMap((page) => page.items),
});

const { data } = useSuspenseInfiniteQuery({
	...usersInfiniteQueryOptions(filters),
	select: selectFlattenedUsers,
});
```

Only add a shared result type when it has real consumers. Otherwise keep the
flattening local to the component.

## Invalidation

Invalidate infinite queries through the same `infiniteQueryOptions` factory.

```ts
await queryClient.invalidateQueries(usersInfiniteQueryOptions(filters));
```

Invalidate the infinite list after mutations that can change list membership,
ordering, active counts, or any aggregate shown alongside the list.
