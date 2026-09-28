# Mutation Usage

## Invalidate With queryOptions

Invalidate with the same `queryOptions` factory the consumers use, so keys are never rebuilt by hand. Return or `await` the promise from `onSuccess`: the mutation stays `isPending` until invalidation finishes.

```ts
const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess(_data, variables, _onMutateResult, context) {
		return Promise.all([
			context.client.invalidateQueries(userQueryOptions(variables.userId)),
			context.client.invalidateQueries(usersQueryOptions(filters)),
		]);
	},
});
```

Use `variables` for the mutated record's query. Also invalidate lists when the mutation can change membership, sorting, filtering, or aggregate counts.

## context.client

The callback `context` (with `context.client`) requires `@tanstack/react-query` v5.89.0+. Check the affected package's installed version if not already known. Signatures:

```ts
onMutate(variables, context) {}
onSuccess(data, variables, onMutateResult, context) {}
onError(error, variables, onMutateResult, context) {}
onSettled(data, error, variables, onMutateResult, context) {}
```

On older or unknown versions, close over `useQueryClient()` instead; there is no need to stop and ask:

```ts
const queryClient = useQueryClient();

const updateUserMutation = useMutation({
	mutationFn: updateUser,
	onSuccess() {
		return queryClient.invalidateQueries(usersQueryOptions(filters));
	},
});
```

## Optimistic Updates

Use only when the UI needs immediate feedback and rollback is clear. Use the same factory for every step: cancel, snapshot, write, roll back, invalidate.

```ts
const updateUserStatusMutation = useMutation({
	mutationFn: updateUserStatus,
	async onMutate(variables, context) {
		const options = usersQueryOptions(filters);
		await context.client.cancelQueries(options);

		const previousUsers = context.client.getQueryData(options.queryKey);

		context.client.setQueryData(options.queryKey, (previous) => {
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
	onError(_error, _variables, onMutateResult, context) {
		context.client.setQueryData(usersQueryOptions(filters).queryKey, onMutateResult?.previousUsers);
	},
	onSettled(_data, _error, _variables, _onMutateResult, context) {
		return context.client.invalidateQueries(usersQueryOptions(filters));
	},
});
```

`onSettled` runs after success and failure. Pick one place for failure handling: `onError`, or an `if (error)` branch in `onSettled` when it belongs with shared cleanup.

## Callback Ownership

Keep cache synchronization in the mutation options. Use `mutate(variables, { onSuccess })` only for local UI effects of that specific call.
