# Mutation State Usage

Use mutation state APIs only when UI outside the component that owns `useMutation` must observe it. Otherwise read state from the `useMutation` result.

## mutationOptions

Use `mutationOptions` when a mutation needs a reusable `mutationKey`, shared defaults, or external observation.

```ts
export function updateUserRoleMutationOptions() {
	return mutationOptions({
		mutationKey: ["users", "update-role"],
		mutationFn: updateUserRole,
	});
}
```

Pass it directly when no callbacks are needed. Otherwise spread it and add callbacks at the consumer, which owns cache sync, navigation, and form errors.

```ts
const updateUserRoleMutation = useMutation({
	...updateUserRoleMutationOptions(),
	onSettled(_data, _error, _variables, _onMutateResult, context) {
		return context.client.invalidateQueries(usersQueryOptions(filters));
	},
});
```

## mutationKey

- Do not add `mutationKey` by default, only when code outside the owner must group, count, or inspect mutations.
- Read the key from the `mutationOptions` factory; never write parallel key arrays in observers.
- Key filters match by prefix. Add `exact: true` to match only the full key.

```ts
const pendingRoleMutations = useIsMutating({
	mutationKey: updateUserRoleMutationOptions().mutationKey,
	status: "pending",
});
```

## useMutationState

Use it when the UI needs mutation details, not just a count. It returns an array, since several mutations can match.

- `mutationKey` picks the mutation family. Make it more precise before reaching for `predicate`.
- `predicate` handles only filtering that key and status cannot express.
- `select` returns just the fields the UI needs; selecting the whole mutation over-subscribes.
- `variables` are not inferred from the key, so cast them locally.

```ts
type UpdateUserRoleVariables = { data: { userId: string } };

const pendingRoleUpdates = useMutationState({
	filters: {
		mutationKey: updateUserRoleMutationOptions().mutationKey,
		status: "pending",
		predicate(mutation) {
			const variables = mutation.state.variables as UpdateUserRoleVariables | undefined;
			return variables?.data.userId === userId;
		},
	},
	select(mutation) {
		return mutation.state.submittedAt;
	},
});
```
