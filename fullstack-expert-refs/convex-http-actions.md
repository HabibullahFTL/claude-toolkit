# Convex HTTP Actions — Full Setup

<!--
  WHEN TO READ THIS FILE:
  - Setting up HTTP actions for the first time (Scenario A bootstrap).
  - Adding a new HTTP action route or handler group.
  - User asks to modify CORS configuration or JSON response helpers.
  DO NOT read for Convex query/mutation/action work — this is only for HTTP endpoints.
-->

---

## File Structure

```
convex/
  http.ts                         ← route registry only — no logic inline
  httpActions/
    index.ts                      ← aggregates all handler groups
    utils.httpActions.ts          ← shared helpers: CORS headers, JSON response builders
    auth/
      auth.httpActions.ts
    notifications/
      notifications.httpActions.ts
    convexStorage/
      image.ts
```

---

## `convex/httpActions/utils.httpActions.ts` — shared helpers

```ts
export const CORS_HEADERS = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, PATCH, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
};

// CORS headers are merged into every JSON response — not just OPTIONS preflight.
// The browser checks for CORS headers on the actual response too, not only the preflight.
const JSON_HEADERS = {
  'Content-Type': 'application/json',
  ...CORS_HEADERS,
};

const jsonResponse = (
  success: boolean,
  message: string,
  status: number,
  data: unknown = null,
) =>
  new Response(JSON.stringify({ success, message, data }), {
    status,
    headers: JSON_HEADERS,
  });

export const successJsonResponseForHttpAction = (
  status: number,
  message: string,
  data: unknown = null,
) => jsonResponse(true, message, status, data);

export const errorJsonResponseForHttpAction = (
  status: number,
  message: string,
) => jsonResponse(false, message, status, null);
```

---

## `convex/httpActions/index.ts` — aggregates all handler groups

```ts
import { authHttpActions } from './auth/auth.httpActions';
import { convexStorageHttpActions } from './convexStorage/image';
// import more handler groups as needed

export const httpActionsHandlers = {
  auth: authHttpActions,
  convexStorage: convexStorageHttpActions,
};
```

---

## `convex/http.ts` — route registry only

```ts
import { httpRouter } from 'convex/server';
import { httpActionsHandlers } from './httpActions';
import { CORS_HEADERS } from './httpActions/utils.httpActions';
import { httpAction } from './_generated/server';

const http = httpRouter();

// ==================== [CORS preflight] =================
// Register OPTIONS for every path that receives cross-origin POST/PATCH/DELETE requests.
const PREFLIGHT_PATHS = ['/api/auth/login', '/api/auth/signup'];
for (const path of PREFLIGHT_PATHS) {
  http.route({
    path,
    method: 'OPTIONS',
    handler: httpAction(
      async () => new Response(null, { status: 204, headers: CORS_HEADERS }),
    ),
  });
}

// ==================== [Auth] =================
http.route({
  path: '/api/auth/login',
  method: 'POST',
  handler: httpActionsHandlers.auth.login,
});
http.route({
  path: '/api/auth/signup',
  method: 'POST',
  handler: httpActionsHandlers.auth.signup,
});

// =================== [Convex Storage] =================
http.route({
  path: '/convex/storage/get-image',
  method: 'GET',
  handler: httpActionsHandlers.convexStorage.getImageContent,
});

// Add more routes here following the same pattern

export default http;
```

---

## Adding a New Handler Group

1. Create `convex/httpActions/<domain>/<domain>.httpActions.ts`
2. Export an object of `httpAction(...)` handlers from that file
3. Import and add to `httpActionsHandlers` in `convex/httpActions/index.ts`
4. Register routes in `convex/http.ts`
5. If the route accepts cross-origin POST/PATCH/DELETE, add it to `PREFLIGHT_PATHS`
