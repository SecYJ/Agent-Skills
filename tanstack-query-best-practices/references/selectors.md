# Select Usage

Use `select` when a consumer only needs part of a query result.

## Choose Selector Shape

Check the app's `package.json` and build config before choosing a selector shape.

React Compiler is enabled when either of these are true:

- `package.json` includes `babel-plugin-react-compiler` in `dependencies` or `devDependencies`.
- `vite.config.ts` imports `babel` from `@rolldown/plugin-babel` and uses `babel({ presets: [reactCompilerPreset()] })` in the `plugins` array.

With React Compiler, inline the selector function inside `useSuspenseQuery`. Do not wrap it in `useCallback` or move it to module scope only for reference stability.

```ts
function ActiveUserCount() {
	const activeUserCountQuery = useSuspenseQuery({
		...usersQueryOptions(filters),
		select: (data: UsersResult) => {
			return data.activeCount;
		},
	});

	// ...
}
```

If React Compiler is not installed, inline `useCallback` inside `useSuspenseQuery` most of the time.

Type the callback's data parameter explicitly because it is not inferred from the later `select` assignment. The dependency array is usually empty when the selector only reads its data parameter; include every component value captured by the selector when it closes over local values.

```ts
function ActiveUserCount() {
	const activeUserCountQuery = useSuspenseQuery({
		...usersQueryOptions(filters),
		select: useCallback((data: UsersResult) => {
			return data.activeCount;
		}, []),
	});

	// ...
}
```

A stable module-scope selector also works, but use it when the user asks for it, multiple consumers share it, or keeping it outside the component improves clarity.

```ts
const selectActiveUserCount = (data: UsersResult) => {
	return data.activeCount;
};

function ActiveUserCount() {
	const activeUserCountQuery = useSuspenseQuery({
		...usersQueryOptions(filters),
		select: selectActiveUserCount,
	});

	// ...
}
```

When a consumer needs multiple values from the same query, return one selected object and destructure it.

```ts
function UserSummary() {
	const { totalCount, activeCount } = useSuspenseQuery({
		...usersQueryOptions(filters),
		select: (data: UsersResult) => {
			return {
				totalCount: data.totalCount,
				activeCount: data.activeCount,
			};
		},
	}).data;

	// ...
}
```
