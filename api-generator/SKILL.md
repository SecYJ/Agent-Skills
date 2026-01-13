---
name: api-generator
description: Generate API and queryOptions based on given md files or a set of url and method for creating API.
---

## When to use this skill

- Use this skill when the user **explicitly asks** to create API functions from **given md docs** (HTTP method, URL, params/body, response shape).
- Use this skill when the user provides a **set of endpoints** (method + URL + params/body) and wants you to generate the API + queryOptions.

## HTTP library + wrapper (what to look for first)

- Check `package.json` to see which HTTP library is installed (commonly **axios** or **ky**).
- Find the project’s HTTP wrapper (examples: `httpClient`, `serverApi`, `apiClient`) and use it instead of calling axios/ky directly.

## Where to put generated APIs

- Put new files under `src/.../(queries/mutation)`:
  - **queries**: read-only / fetching data (usually GET).
  - **mutation**: updating server resources (POST/PUT/PATCH/DELETE).
- If the user did not specify the location, ask: **“Where should I create this API at?”**
- Keep the folder naming consistent with the nearest existing module (some modules use `mutation/` vs `mutations/`).

## How to create `queryOptions`

- Place `queryOptions` in the same hierarchy as the API, under `src/.../(queries/mutation)`.
- Put **all queryOptions** in a `utils.ts` file and **export them from `utils.ts`**.
- Every `queryOptions` must be **wrapped in a function** (no exported constants).
- The **query key** should follow the API function name:
  - Use an array key like `['getSomething', ...params]` so params participate in caching.
  - If the API has params, include them in the key in the same order as the function args.

See [the api guide](references/api.md) for details.
See [the queryOptions guide](references/queryOptions.md) for details.

