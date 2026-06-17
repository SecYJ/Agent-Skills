# Dependent Query Usage

Use `enabled` when a query must wait for a required input.

```ts
import { queryOptions, useQuery } from "@tanstack/react-query";

import { getUser } from "./api";

export const userQueryOptions = (userId: string) =>
	queryOptions({
		queryKey: ["users", userId],
		queryFn: () => getUser(userId),
	});

const UserDetails = ({ userId }: { userId?: string }) => {
	const userQuery = useQuery({
		...userQueryOptions(userId ?? ""),
		enabled: Boolean(userId),
	});

	return <UserProfile user={userQuery.data} />;
};
```

## Dependent On Another Query

Include the derived input in the dependent query key and keep the query disabled
until the input exists.

```ts
const currentUserQuery = useSuspenseQuery(currentUserQueryOptions());
const managerId = currentUserQuery.data.managerId;

const managerQuery = useQuery({
	...userQueryOptions(managerId),
	enabled: Boolean(managerId),
});
```

Use `useQuery` for a disabled query because `useSuspenseQuery` is intended for
queries that can run immediately.

```ts
const { userId } = Route.useParams();

const { data } = useSuspenseQuery(userQueryOptions(userId));
```

## Modal And Drawer Preview Queries

Gate preview queries behind the UI state that makes them visible.

```ts
const userPreviewQuery = useQuery({
	...userQueryOptions(selectedUserId ?? ""),
	enabled: open && Boolean(selectedUserId),
});
```
