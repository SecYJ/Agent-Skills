# Infinite Query Usage

Use `infiniteQueryOptions` for cursor- or page-based lists. Put every input that changes list identity (filters, search, sort, page size) in the key, but never `pageParam`, since all pages live in one cache entry.

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

## Consumers

Use `useSuspenseInfiniteQuery` when the data can load immediately, and `useInfiniteQuery` when it can be disabled.

```ts
const usersQuery = useSuspenseInfiniteQuery(usersInfiniteQueryOptions(filters));
const users = usersQuery.data.pages.flatMap((page) => page.items);

const drawerUsersQuery = useInfiniteQuery({ ...usersInfiniteQueryOptions(filters), enabled: open });
const drawerUsers = drawerUsersQuery.data?.pages.flatMap((page) => page.items) ?? [];
```

When several consumers need flattened data, share a `select` that keeps `pages` and `pageParams`:

```ts
function selectFlattenedUsers(data: InfiniteData<UsersPageResult, string | undefined>) {
	return {
		...data,
		users: data.pages.flatMap((page) => page.items),
	};
}
```

Invalidate through the same factory: `queryClient.invalidateQueries(usersInfiniteQueryOptions(filters))`.
