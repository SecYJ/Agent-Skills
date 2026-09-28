# Usage With TanStack Router

With TanStack Router, prefer `useSuspenseQuery` for route data. Build concrete query options in route `context`, seed the cache in `loader`, and read them in the component. Loader and component then share one cache entry, and the component never rebuilds options from route inputs.

When a component needs more than one route query, use `useSuspenseQueries` instead of several `useSuspenseQuery` calls (see query-options.md).

Select search-param inputs with `loaderDeps` and combine them with path params in `context`.

```tsx
export const Route = createFileRoute("/dashboard/$dashboardId")({
	validateSearch: dashboardSearchSchema,
	loaderDeps({ search: { asOf } }) {
		return { asOf };
	},
	context({ params, deps }) {
		return {
			dashboardQueryOptions: dashboardQueryOptions(params.dashboardId, deps),
		};
	},
	loader({ context }) {
		context.queryClient.ensureQueryData(context.dashboardQueryOptions);
	},
	component: DashboardPage,
});

function DashboardPage() {
	const { dashboardQueryOptions } = Route.useRouteContext();
	const { data } = useSuspenseQuery(dashboardQueryOptions);

	return <Dashboard data={data} />;
}
```

## Seeding in Loaders

In most cases, call `ensureQueryData` in the loader without `await`, `return`, or `void`. The loader starts the fetch without blocking page rendering, and `useSuspenseQuery` suspends on the same in-flight request.

Await only when the route must not render until the data exists. Then await all of it together:

```tsx
async loader({ context }) {
	await Promise.all([
		context.queryClient.ensureQueryData(context.dashboardQueryOptions),
		context.queryClient.ensureQueryData(context.dashboardSummaryQueryOptions),
	]);
},
```
