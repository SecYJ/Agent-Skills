# Mutation State Usage

Use mutation state APIs only when UI outside the component that owns `useMutation` needs to observe mutation activity. Keep ordinary mutation state local to the `useMutation` result.

## mutationOptions

Use `mutationOptions` when a mutation needs a reusable `mutationKey`, shared defaults, or external observation through `useMutationState` or `useIsMutating`.

```ts
import { mutationOptions } from "@tanstack/react-query";

export const updateUserRoleMutationOptions = () =>
	mutationOptions({
		mutationKey: ["users", "update-role"],
		mutationFn: updateUserRole,
	});
```

When no consumer-level callback is needed, call the options factory directly in `useMutation`.

```ts
const updateUserRoleMutation = useMutation(updateUserRoleMutationOptions());
```

When the consumer defines `onSuccess`, `onError`, `onMutate`, or `onSettled`, spread the shared mutation options and define the callback at the consumer.

```ts
const updateUserRoleMutation = useMutation({
	...updateUserRoleMutationOptions(),
	onSettled: async () => {
		await queryClient.invalidateQueries(usersQueryOptions(filters));
	},
});
```

This keeps the shared `mutationKey` and `mutationFn` in one place while leaving cache synchronization, navigation, form errors, and local side effects with the consumer that owns them.

## mutationKey

Do not add `mutationKey` by default. Add it when code needs to group, count, inspect, or select mutation state outside the mutation owner.

Do not define parallel mutation key arrays in consumers. When a mutation key is needed, read it from the `mutationOptions` factory that owns the mutation.

```ts
const pendingRoleMutations = useIsMutating({
	mutationKey: updateUserRoleMutationOptions().mutationKey,
	status: "pending",
});
```

`mutationKey` filters are partial by default, so a shorter key matches mutations keyed under that prefix. Use `exact: true` when only the full key should match. If code needs a broader grouped key, make that key part of a shared `mutationOptions` factory instead of creating an ad hoc array at the observer.

```ts
const pendingExactRoleMutations = useIsMutating({
	mutationKey: updateUserRoleMutationOptions().mutationKey,
	exact: true,
	status: "pending",
});
```

## useMutationState

Use `useMutationState` when the UI needs mutation details, not just a count. It returns an array because multiple matching mutations can be pending or completed at the same time.

```ts
const pendingRoleVariables = useMutationState({
	filters: {
		mutationKey: updateUserRoleMutationOptions().mutationKey,
		status: "pending",
	},
	select: (mutation) => mutation.state.variables,
});
```

Prefer all three pieces when reading mutation state:

- `mutationKey`: choose the mutation family to observe.
- `predicate`: narrow matches that cannot be expressed by key or status.
- `select`: return only the fields the UI needs.

```ts
type UpdateUserRoleVariables = {
	data: {
		userId: string;
	};
};

const pendingRoleUpdateForUser = useMutationState({
	filters: {
		mutationKey: updateUserRoleMutationOptions().mutationKey,
		status: "pending",
		predicate: (mutation) => {
			const variables = mutation.state.variables as UpdateUserRoleVariables | undefined;

			return variables?.data.userId === userId;
		},
	},
	select: (mutation) => ({
		submittedAt: mutation.state.submittedAt,
		variables: mutation.state.variables as UpdateUserRoleVariables | undefined,
	}),
});
```

Keep `predicate` for true cache-entry filtering. Do not use it to compensate for vague mutation keys. If a different screen naturally needs a different group of mutations, make the `mutationKey` more precise first.

When `predicate` or `select` reads `mutation.state.variables`, narrow the variables shape locally. `useMutationState` observes the mutation cache and does not automatically infer variables from the matching `mutationKey`.

Keep `select` small. Selecting the full mutation state subscribes the component to more changes than it usually needs.
