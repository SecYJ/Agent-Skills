# Mutation Usage

Use `useMutation` to mutate data on the server.

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
});
```

## Invalidate With queryOptions

Prefer invalidating queries with the same `queryOptions` factory used by query consumers.

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess: async (_1, _2, _3, context) => {
		await context.client.invalidateQueries(usersQueryOptions(filters));
	},
});
```

This keeps the mutation aligned with the query's single source of truth. The mutation does not need to rebuild the query key by hand, so query consumers, route loaders, prefetching, and invalidation all point at the same query definition.

## Check context.client Support

Get the installed `@tanstack/react-query` package version before using `context.client` in mutation callbacks, for example with `pnpm list @tanstack/react-query --depth 0` from the package root. Mutation callback context, including `context.client`, is supported in v5.89.0 and later.

The context parameter is fourth for `onSuccess` and `onError`, and fifth for `onSettled`:

```ts
onSuccess: (_data, _variables, onMutateResult, context) => context.client.invalidateQueries(options),
onError: (_data, _variables, onMutateResult, context) => context.client.invalidateQueries(options),
onSettled: (_data, _error, _variables, onMutateResult, context) => context.client.invalidateQueries(options),
```

For versions earlier than v5.89.0, initialize the client with `useQueryClient` and close over it in the callbacks instead:

```ts
const queryClient = useQueryClient();

const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess: async () => {
		await queryClient.invalidateQueries(usersQueryOptions(filters));
	},
});
```

## Keep isPending True During Invalidation

React Query keeps `isPending` true while a promise returned from `onSuccess` is still pending. Use this when the mutation should stay pending until invalidation finishes.

Use `async`/`await`:

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess: async (_1, _2, _3, context) => {
		await context.client.invalidateQueries(usersQueryOptions(filters));
	},
});
```

Or return the promise directly:

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess: (_1, _2, _3, context) => context.client.invalidateQueries(usersQueryOptions(filters)),
});
```

Both forms tell React Query to wait for the invalidation promise to resolve before the mutation leaves its pending state.

## Pass Variables Into queryOptions

Use mutation variables when the invalidated query depends on the mutated record.

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess: async (_1, variables, _3, context) => {
		await context.client.invalidateQueries(userQueryOptions(variables.userId));
	},
});
```

Invalidate the list query too when the mutation can change list membership, sorting, filters, or aggregate counts.

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess: async (_1, variables, _3, context) => {
		await Promise.all([
			context.client.invalidateQueries(userQueryOptions(variables.userId)),
			context.client.invalidateQueries(usersQueryOptions(filters)),
		]);
	},
});
```

## Optimistic Update With queryOptions

Use optimistic updates only when the UI needs the immediate feedback and rollback behavior is clear.

```ts
const updateUserStatusMutation = useMutation({
	mutationFn: updateUserStatus,
	onMutate: async (variables, context) => {
		await context.client.cancelQueries(usersQueryOptions(filters));

		const previousUsers = context.client.getQueryData(usersQueryOptions(filters).queryKey);

		context.client.setQueryData(usersQueryOptions(filters).queryKey, (previous) => {
			if (!previous) return previous;

			return {
				...previous,
				items: previous.items.map((user) =>
					user.id === variables.userId ? { ...user, status: variables.status } : user,
				),
			};
		});

		return { previousUsers };
	},
	onError: (_1, _2, onMutateResult, context) => {
		context.client.setQueryData(usersQueryOptions(filters).queryKey, onMutateResult?.previousUsers);
	},
	onSettled: async (_1, error, _3, _4, context) => {
		if (error) {
			// do something
		}

		await context.client.invalidateQueries(usersQueryOptions(filters));
	},
});
```

The same `queryOptions` factory should be used for every cache operation in the optimistic flow: cancel the active query, snapshot the old data, write the optimistic data, rollback on error, and invalidate after settlement.

Use `onSettled` for work that should run after both successful and failed mutations, such as query invalidation. The `error` argument tells the callback whether the mutation failed, so one callback can handle shared cleanup and still branch for error-specific behavior when needed. If all error handling, including optimistic rollback, is handled inside the `onSettled` error branch, a separate `onError` callback is not needed. The `onMutateResult` argument is the value returned from the `onMutate` callback, such as the rollback snapshot in the example above.

Choose one owner for failure handling. Use `onError` when the failure path is separate and clearer there; use the `onSettled` error branch when shared settlement work and failure handling belong together.

## Callback Ownership

Put server cache synchronization in the mutation options that own the mutation behavior. Use consumer-level `mutate(variables, { onSuccess })` callbacks only for local UI effects tied to that specific call.
