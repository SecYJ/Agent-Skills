# Usage With TanStack Router

## Prefer Suspense Query Consumers

For route-owned page data, use `useSuspenseQuery` and let the route loader seed
the TanStack Query cache. The component should consume the concrete query
options exposed by route context instead of rebuilding them from route inputs.

## Prefer A Plain `ensureQueryData` Call In Loaders

Usually call `queryClient.ensureQueryData(...)` directly without returning it,
awaiting it, or using `void`. If a loader must await query work, use
`await Promise.all([...])`.

## Build Concrete Query Options From `loaderDeps`

When a query depends on search params, select the relevant values with
`loaderDeps` and use them with path params to build the query options in
`context`. Reuse those options in the loader and component so both request the
same cache entry.

```tsx
import { useSuspenseQuery } from "@tanstack/react-query";
import { createFileRoute } from "@tanstack/react-router";

import { dashboardQueryOptions } from "@/features/dashboard/services/queries";
import { dashboardSearchSchema } from "@/features/dashboard/services/schemas";

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

When a loader must await multiple queries, group them with `Promise.all`:

```tsx
async loader({ context }) {
	await Promise.all([
		context.queryClient.ensureQueryData(context.dashboardQueryOptions),
		context.queryClient.ensureQueryData(context.dashboardSummaryQueryOptions),
	]);
},
```
