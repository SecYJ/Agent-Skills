# Usage With TanStack Router

## Prefer Suspense Query Consumers

For route-owned page data, use `useSuspenseQuery` and let the route loader seed
the TanStack Query cache. The component should consume the concrete query
options exposed by route context instead of rebuilding them from route inputs.

```tsx
function DashboardPage() {
	const { dashboardQueryOptions } = Route.useRouteContext();
	const { data } = useSuspenseQuery(dashboardQueryOptions);

	return <Dashboard data={data} />;
}
```

## Handle Every `ensureQueryData` Promise Deliberately

Never leave an `ensureQueryData` promise floating without expressing whether
the Router should wait for it:

- For blocking data, return or await the promise so TanStack Router owns its
  pending and error lifecycle. Returning the promise does not require an
  `async` loader.
- For multiple blocking queries, return `Promise.all([...])` or await it inside
  an `async` loader.
- For intentional non-blocking loading, prefix the call with `void`. Use this
  only when the component can pick up the query and handle its pending or error
  state.

Do not write a bare `queryClient.ensureQueryData(...)` statement. It hides
whether the missing return was intentional.

```tsx
loader: ({ context }) => {
	void context.queryClient.ensureQueryData(
		context.dashboardActivityQueryOptions,
	);

	return Promise.all([
		context.queryClient.ensureQueryData(context.dashboardQueryOptions),
		context.queryClient.ensureQueryData(context.dashboardSummaryQueryOptions),
	]);
},
```

## Build Concrete Query Options From `loaderDeps`

When a query depends on search params, select the data dependencies with
`loaderDeps`. Use those values, together with path params, to build the concrete
query options in the route's `context` callback. The loader and component must
then reuse that options object.

This prevents the loader and component from drifting apart when a filter or
other query input changes. Without this shared object, the loader can prefetch
one cache entry while the component requests another.

```tsx
import { useSuspenseQuery } from "@tanstack/react-query";
import { createFileRoute } from "@tanstack/react-router";

import { dashboardQueryOptions } from "@/features/dashboard/services/queries";

type DashboardSearch = {
	asOf?: string;
};

export const Route = createFileRoute("/dashboard/$dashboardId")({
	validateSearch: (search): DashboardSearch => ({
		asOf: typeof search.asOf === "string" ? search.asOf : undefined,
	}),
	loaderDeps: ({ search: { asOf } }) => ({ asOf }),
	context: ({ params, deps }) => ({
		dashboardQueryOptions: dashboardQueryOptions(params.dashboardId, deps),
	}),
	loader: ({ context }) =>
		context.queryClient.ensureQueryData(context.dashboardQueryOptions),
	component: DashboardPage,
});

function DashboardPage() {
	const { asOf } = Route.useSearch();
	const { dashboardQueryOptions } = Route.useRouteContext();
	const { data } = useSuspenseQuery(dashboardQueryOptions);

	return <Dashboard data={data} selectedDate={asOf} />;
}
```

Each route API has one responsibility:

- `validateSearch` parses and types the route's search params.
- `loaderDeps` selects only the search values that affect fetched data.
- `context` turns `params` and `loaderDeps` into concrete query options.
- `loader` returns blocking `ensureQueryData` work and explicitly marks
  intentional non-blocking work with `void`.
- `useRouteContext` gives the component the same options for
  `useSuspenseQuery`.
- `useSearch` gives the component search values needed for rendering or other
  UI logic.

The shared feature-level `queryOptions` factory remains the reusable query
definition. Route context owns the concrete options for one matched route.

## Keep Raw Route Values Out Of Route Context

Expose concrete query options through route context, not raw path params,
search params, or `loaderDeps` values. Components already have reactive,
type-safe Router APIs for those values:

```tsx
function DashboardToolbar() {
	const { dashboardId } = Route.useParams();
	const { asOf } = Route.useSearch();

	return <Toolbar dashboardId={dashboardId} selectedDate={asOf} />;
}
```

Use `Route.useSearch()` when search params affect labels, selected controls,
links, or other UI behavior. Use `Route.useParams()` when the UI needs path
params. Do not add `dashboardId`, `asOf`, or a raw `deps` object to route context
as an alternative way to access Router state.

It is fine for the component to read search params for UI logic while its query
uses the options from route context. Do not use those values to reconstruct the
query options a second time.

## Inherit Query Options From Parent Routes

Route context is inherited. A descendant can consume query options created by
its parent routes alongside its own route's options:

```tsx
function DashboardWidget() {
	const { currentUserQueryOptions, dashboardQueryOptions } =
		Route.useRouteContext();

	const currentUserQuery = useSuspenseQuery(currentUserQueryOptions);
	const dashboardQuery = useSuspenseQuery(dashboardQueryOptions);

	return (
		<Widget
			user={currentUserQuery.data}
			dashboard={dashboardQuery.data}
		/>
	);
}
```

Add query options to a parent only when the data belongs to that route subtree.
Do not turn root route context into a registry for every query in the app.

The route-level `context` callback is a newer TanStack Router API. Confirm that
the installed `@tanstack/react-router` version supports it before applying this
pattern. See [Reliable Query Prefetching with TanStack Router](https://tkdodo.eu/blog/reliable-query-prefetching-with-tanstack-router)
for the original explanation of the loader/component drift problem.
