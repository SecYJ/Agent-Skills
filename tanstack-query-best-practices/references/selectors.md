# Select Usage

Use `select` when a consumer only needs part of a query result.

## Choose Selector Shape

Check the app's `package.json` and build config before choosing a selector shape.

React Compiler is enabled when either of these are true:

- `package.json` includes `babel-plugin-react-compiler` in `dependencies` or `devDependencies`.
- `vite.config.ts` imports `babel` from `@rolldown/plugin-babel` and uses `babel({ presets: [reactCompilerPreset()] })` in the `plugins` array.

### With React Compiler

Inline the selector function inside `useSuspenseQuery`. Do not wrap it in `useCallback` or move it to module scope only for reference stability.

Do not annotate the selector's `data` parameter. Its type is inferred from the spread query options (for example `...usersQueryOptions(filters)`), so leave it bare and do not import the result type solely for that annotation.

```ts
function ActiveUserCount() {
	const activeUserCountQuery = useSuspenseQuery({
		...usersQueryOptions(filters),
		select: (data) => {
			return data.activeCount;
		},
	});

	// ...
}
```

This inference only holds for a single `useSuspenseQuery` or `useQuery` with spread query options. With `useQueries` and `useSuspenseQueries`, the per-query `select` callback's `data` parameter is not inferred from the queries array (it falls back to `any`), so annotate it explicitly even when React Compiler is enabled. This is a long-standing TypeScript inference limitation tracked in TanStack Query issue [#3994](https://github.com/TanStack/query/issues/3994).

```ts
function ActiveUserCount() {
	const [activeUserCountQuery] = useSuspenseQueries({
		queries: [
			{
				...usersQueryOptions(filters),
				// Annotate `data` even under React Compiler; useQueries and
				// useSuspenseQueries do not infer the select parameter.
				select: (data: UsersResult) => data.activeCount,
			},
		],
	});

	// ...
}
```

### Without React Compiler

If React Compiler is not installed, inline `useCallback` inside `useSuspenseQuery` most of the time.

Type the callback's `data` parameter explicitly here, because `useCallback`'s generic breaks the inference from the spread query options that the React Compiler case relies on. Without the annotation `data` would be untyped. The dependency array is usually empty when the selector only reads its data parameter; include every component value captured by the selector when it closes over local values.

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

### Module-Scope Selector

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

### Multiple Selected Values

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
