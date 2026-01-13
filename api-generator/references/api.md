## API examples

### Query (GET) example

- **Location**: `src/.../queries/getAgentDetails.ts`
- **Rule**: use the project HTTP wrapper (ex: `serverApi`) instead of calling ky/axios directly.

```ts
import { serverApi } from 'src/shared/context/ky';
import { TApiSuccess } from 'src/shared/services/types';

type TGetAgentDetailsResponse = {
  agentId: string;
  name: string;
};

export const getAgentDetails = (agentId: string) => {
  return serverApi
    .get('app/team/agent/details', {
      searchParams: { agentId },
    })
    .json<TApiSuccess<TGetAgentDetailsResponse>>();
};
```

### Mutation (POST) example

- **Location**: `src/.../mutation/updateAgentNickname.ts`

```ts
import { serverApi } from 'src/shared/context/ky';
import { TApiSuccess } from 'src/shared/services/types';

type TUpdateAgentNicknameBody = {
  agentId: string;
  nickname: string;
};

export const updateAgentNickname = (body: TUpdateAgentNicknameBody) => {
  return serverApi
    .post('app/team/agent/nickname', {
      json: body,
    })
    .json<TApiSuccess>();
};
```

