## `queryOptions` examples

### Pattern

- **Location**: keep it alongside APIs under `src/.../queries/`.
- **Filename**: **`utils.ts`**
- **Export**: export **all queryOptions** from `utils.ts`.
- **Rule**: every queryOptions must be a **function**.
- **Rule**: `queryKey` follows the **API function name** (and include params).

### Example (`utils.ts`)

```ts
import { queryOptions } from '@tanstack/react-query';
import { getAgentDetails } from './getAgentDetails';

export const getAgentDetailsQueryOptions = (agentId: string) => {
  return queryOptions({
    queryKey: ['getAgentDetails', agentId],
    queryFn: () => getAgentDetails(agentId),
  });
};
```

