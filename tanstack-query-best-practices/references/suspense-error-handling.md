# Suspense Error Handling

When a route page uses `useSuspenseQuery`, let Suspense handle loading and let
the nearest route error boundary handle thrown query errors.

```tsx
import { useSuspenseQuery } from "@tanstack/react-query";
import { ErrorComponent, createFileRoute, useRouter } from "@tanstack/react-router";

import { usersQueryOptions } from "@/features/users/services/queries";

export const Route = createFileRoute("/users")({
	context: () => ({
		usersQueryOptions: usersQueryOptions(),
	}),
	loader: ({ context }) =>
		context.queryClient.ensureQueryData(context.usersQueryOptions),
	errorComponent: UsersError,
	component: UsersPage,
});

function UsersPage() {
	const { usersQueryOptions } = Route.useRouteContext();
	const { data } = useSuspenseQuery(usersQueryOptions);

	return <UsersList users={data.users} />;
}

const UsersError = ({ error }: { error: Error }) => {
	const router = useRouter();

	return (
		<div>
			<ErrorComponent error={error} />
			<button type="button" onClick={() => router.invalidate()}>
				Retry
			</button>
		</div>
	);
};
```

Use `router.invalidate()` for route-level retry buttons because it re-runs
active route loaders and gives prefetched queries another chance to seed the
cache.

## Suspense Fallback

Wrap Suspense query components in a Suspense boundary when the route or parent
layout does not already provide one.

```tsx
import { Suspense } from "react";

const UsersRouteContent = () => (
	<Suspense fallback={<UsersSkeleton />}>
		<UsersPage />
	</Suspense>
);
```

## Reset Query Errors Outside Route Boundaries

Use `QueryErrorResetBoundary` when a Suspense query is handled by a local React
error boundary instead of a TanStack Router route error boundary.

```tsx
import { QueryErrorResetBoundary } from "@tanstack/react-query";
import { ErrorBoundary } from "react-error-boundary";

const UsersBoundary = () => (
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
);
```

Keep reset ownership close to the boundary that catches the error. Route errors
should usually retry through the router. Local widget errors should reset
through `QueryErrorResetBoundary`.

## Avoid Local Loading Branches

Do not add `isLoading` branches around `useSuspenseQuery`. If a page needs a
loading state, put it in the route pending component or a Suspense fallback.

```tsx
const UsersPage = () => {
	const { usersQueryOptions } = Route.useRouteContext();
	const { data } = useSuspenseQuery(usersQueryOptions);

	return <UsersList users={data.users} />;
};
```
