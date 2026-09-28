# Suspense Error Handling

Suspense handles loading for `useSuspenseQuery`; the nearest error boundary handles thrown errors. Connect Query's reset to whichever boundary catches the error. Do not add a second boundary just to reset.

## Loading

Do not add `isLoading` branches around `useSuspenseQuery`. Rely on the route `pendingComponent` or a parent `<Suspense fallback>`; add one only if none exists.

## Route Error Boundaries

Reset Query's error boundary when the route `errorComponent` mounts, so failed queries can refetch on the next render, including after navigating away and back. Retry with `router.invalidate()` to rerun loaders and reset the route boundary.

```tsx
export const Route = createFileRoute("/users")({
	// context, loader, component: see router.md
	errorComponent: UsersError,
});

function UsersError({ error }: ErrorComponentProps) {
	const router = useRouter();
	const queryErrorResetBoundary = useQueryErrorResetBoundary();

	useEffect(() => {
		queryErrorResetBoundary.reset();
	}, [queryErrorResetBoundary]);

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

## Local Error Boundaries

When a local React error boundary catches the error, wire `QueryErrorResetBoundary`'s `reset` into its `onReset`.

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
