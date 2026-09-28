# Dependent Query Usage

When a query must wait for a required input, return `skipToken` as the `queryFn` until the input exists. The query stays disabled, and `userId` narrows to `string` without a runtime guard. Include the input in the key.

```ts
import { queryOptions, skipToken } from "@tanstack/react-query";

export function userQueryOptions(userId?: string) {
	return queryOptions({
		queryKey: ["users", userId],
		queryFn: userId
			? () => getUser(userId)
			: skipToken,
	});
}
```

`skipToken` does not work with `useSuspenseQuery`. Use `useQuery` for queries that can be skipped, and `useSuspenseQuery` only when the inputs are already available.

```ts
// Waits for another query's result
const managerId = useSuspenseQuery(currentUserQueryOptions()).data.managerId;
const managerQuery = useQuery(userQueryOptions(managerId));
```

For conditions that are not query inputs, such as UI visibility, add `enabled` on top of the factory:

```ts
// Runs only while a modal or drawer is open
const previewQuery = useQuery({
	...userQueryOptions(selectedUserId),
	enabled: open,
});
```
