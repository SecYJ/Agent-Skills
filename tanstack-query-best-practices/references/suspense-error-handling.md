# Suspense Error Handling

Let Suspense handle loading for `useSuspenseQuery`. Let the nearest route error
boundary handle thrown query errors.

## Route Error Boundaries

```tsx
export const Route = createFileRoute("/users")({
	context() {
		return { usersQueryOptions: usersQueryOptions() };
	},
	loader({ context }) {
		context.queryClient.ensureQueryData(context.usersQueryOptions);
	},
	errorComponent: UsersError,
	component: UsersPage,
});

function UsersPage() {
	const { usersQueryOptions } = Route.useRouteContext();
	const { data } = useSuspenseQuery(usersQueryOptions);

	return <UsersList users={data.users} />;
}

function UsersError({ error }: { error: Error }) {
	const router = useRouter();

	return (
		<div>
			<ErrorComponent error={error} />
			<button type="button" onClick={() => router.invalidate()}>
				Retry
			</button>
		</div>
	);
}
```

Use `router.invalidate()` for route-level retries so active loaders run again.

## Loading Fallback

Do not add `isLoading` branches around `useSuspenseQuery`. Use the route pending
component or a Suspense fallback when a parent does not already provide one.

```tsx
<Suspense fallback={<UsersSkeleton />}>
	<UsersPage />
</Suspense>
```

## Local Error Boundaries

Use `QueryErrorResetBoundary` when a local React error boundary, rather than a
route error boundary, catches a Suspense query error.

```tsx
<QueryErrorResetBoundary>
	{({ reset }) => (
		<ErrorBoundary
			onReset={reset}
			fallbackRender={({ error, resetErrorBoundary }) => (
				<div>
					<p>{error.message}</p>
					<button type="button" onClick={resetErrorBoundary}>
						Retry
					</button>
				</div>
			)}
		>
			<UsersPage />
		</ErrorBoundary>
	)}
</QueryErrorResetBoundary>
```

Keep reset ownership with the boundary that catches the error: route errors use
the router, while local errors use `QueryErrorResetBoundary`.
