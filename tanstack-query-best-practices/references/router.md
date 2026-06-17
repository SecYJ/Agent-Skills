# Usage With TanStack Router

## Prefer Suspense Query Consumers

When working with TanStack Router, use `useSuspenseQuery` instead of `useQuery`
for route page data. Route loaders should seed the query cache, and route
components should consume that data through Suspense instead of branching on
`isLoading`.

```tsx
import { useSuspenseQuery } from "@tanstack/react-query";

import { usersQueryOptions } from "@/features/users/services/queries";

const UsersPage = () => {
	const { data } = useSuspenseQuery(usersQueryOptions());

	return <UsersList users={data.users} />;
};
```

## Seed Query Cache From Route Loaders

Use route loaders to start TanStack Query requests before the route component
renders. Prefer starting the query without `async`/`await` so navigation is not
blocked by the whole route's data fetch.

```tsx
import { createFileRoute } from "@tanstack/react-router";

import { usersQueryOptions } from "@/features/users/services/queries";

export const Route = createFileRoute("/users")({
	loader: ({ context: { queryClient } }) => {
		queryClient.ensureQueryData(usersQueryOptions());
	},
	component: UsersPage,
});
```

Use `revalidateIfStale` when the data source is very important or should
refresh on every navigation. With the usual `staleTime: 0` behavior, this gives
consumers cached data immediately while starting a fresh request in the
background.

```tsx
import { createFileRoute } from "@tanstack/react-router";

import { usersQueryOptions } from "@/features/users/services/queries";

export const Route = createFileRoute("/users")({
	loader: ({ context: { queryClient } }) => {
		queryClient.ensureQueryData({
			...usersQueryOptions(),
			revalidateIfStale: true,
		});
	},
	component: UsersPage,
});
```

Only make the loader `async` and `await` the query when the route must block
before rendering, such as auth checks, permissions, required tenant context, or
third-party integration handshakes.

```tsx
export const Route = createFileRoute("/users/$userId/integrations")({
	loader: async ({ context: { queryClient } }) => {
		await queryClient.ensureQueryData(userIntegrationSessionQueryOptions());
	},
	component: UserIntegrationsPage,
});
```
