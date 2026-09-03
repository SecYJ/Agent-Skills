# Infinite Query Usage

Use `infiniteQueryOptions` for cursor- or page-based lists that load more data
from the same query family.

```ts
export function usersInfiniteQueryOptions(filters: UserFilters) {
	return infiniteQueryOptions({
		queryKey: ["users", "infinite", filters],
		queryFn({ pageParam }) {
			return getUsers({ ...filters, cursor: pageParam });
		},
		initialPageParam: undefined as string | undefined,
		getNextPageParam(lastPage) {
			return lastPage.nextCursor;
		},
	});
}
```

Include every stable input that changes the list identity in the query key,
such as filters, search, sorting, and page size. Do not include `pageParam`;
TanStack Query stores every page in the same infinite-query cache entry.

## Consumers

Use `useSuspenseInfiniteQuery` for route data that can load immediately.

```ts
const usersQuery = useSuspenseInfiniteQuery(
	usersInfiniteQueryOptions(filters),
);
const users = usersQuery.data.pages.flatMap((page) => page.items);
```

Use `useInfiniteQuery` when the query can be disabled.

```ts
const usersQuery = useInfiniteQuery({
	...usersInfiniteQueryOptions(filters),
	enabled: open,
});

const users = usersQuery.data?.pages.flatMap((page) => page.items) ?? [];
```

## Derived Data

When multiple consumers need flattened data, use a shared `select` function.
Keep `pages` and `pageParams` in the result so infinite-query metadata remains
available.

```ts
type UsersPageResult = Awaited<ReturnType<typeof getUsers>>;

function selectFlattenedUsers(data: {
	pages: UsersPageResult[];
	pageParams: unknown[];
}) {
	return {
		...data,
		users: data.pages.flatMap((page) => page.items),
	};
}

const { data } = useSuspenseInfiniteQuery({
	...usersInfiniteQueryOptions(filters),
	select: selectFlattenedUsers,
});
```

## Invalidation

Invalidate through the same infinite-query options factory.

```ts
await queryClient.invalidateQueries(usersInfiniteQueryOptions(filters));
```
