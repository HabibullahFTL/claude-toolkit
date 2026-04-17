---
description: Convex function authoring — middleware wrappers, response helpers, comment tags, schema patterns, file structure
globs: convex/**/*.ts,convex/**/*.tsx
---

# Convex Rules

## Middleware — Required on Every Function

**Never write a bare `query`, `mutation`, or `action`.** Always wrap:

| Wrapper | Import from | Use when |
|---|---|---|
| `convexMiddleware` | `../utils/middlewares/convexMiddleware` | Requires authenticated user |
| `convexPublicMiddleware` | `../utils/middlewares/convexPublicMiddleware` | Public / no auth required |

```ts
// ✅ Correct
export const createItem = mutation(
  convexMiddleware({
    args: { name: v.string() },
    zodSchema: z.object({ name: z.string().min(1) }),
    handler: async (ctx, args, currentUser) => { ... },
  }),
);

// ❌ Wrong
export const createItem = mutation({
  args: { name: v.string() },
  handler: async (ctx, args) => { ... },
});
```

## Comment Tags

Place **directly above** every exported Convex function:

| Tag | Use when |
|---|---|
| `// [USER-FN]: description` | Authenticated, regular user access |
| `// [ADMIN-FN]: description` | Admin role / permission required |
| `// [PUBLIC-FN]: description` | No authentication required |
| `// [INTERNAL-FN]: description` | Convex internal functions (`internalMutation`, `internalAction`) |

Use section dividers when a file has multiple access levels:
```ts
// ==================== [USER-FN] =================

// [USER-FN]: Create item
export const createItem = mutation(convexMiddleware({ ... }));

// * ==================== [ADMIN-FN] =================

// * [ADMIN-FN]: Delete any item
export const adminDeleteItem = mutation(convexMiddleware({ ... }));
```

## Response Helpers — Never Throw

**Never use `throw new Error()` or `ConvexError`.** Always return:

```ts
import { generateConvexSuccessResponse, generateConvexErrorResponse } from '../../utils/common';
import HttpStatusCodes from '../../utils/httpStatusCode';

// Success
return generateConvexSuccessResponse(HttpStatusCodes.OK, 'Items fetched.', data);
return generateConvexSuccessResponse(HttpStatusCodes.CREATED, 'Item created.', newId, meta);

// Error
return generateConvexErrorResponse(HttpStatusCodes.NOT_FOUND, 'Item not found.');
return generateConvexErrorResponse(HttpStatusCodes.UNAUTHORIZED, 'Not authenticated.');
return generateConvexErrorResponse(HttpStatusCodes.INTERNAL_SERVER_ERROR, 'Something went wrong.');
```

Use `HttpStatusCodes` from `convex/utils/httpStatusCode.ts` — **never hardcode numbers** like `400` or `200`.

Wrap every handler in `try/catch` and return `INTERNAL_SERVER_ERROR` for unexpected failures:
```ts
handler: async (ctx, args, currentUser) => {
  try {
    const data = await ctx.db.insert('items', args);
    return generateConvexSuccessResponse(HttpStatusCodes.CREATED, 'Created.', data);
  } catch {
    return generateConvexErrorResponse(HttpStatusCodes.INTERNAL_SERVER_ERROR, 'Something went wrong.');
  }
},
```

## Zod Validation

Every middleware call must include a `zodSchema`. Reuse from `convex/validations/common.ts`:
- `emailZodSchema` — validated email
- `userIdZodSchema` — branded `Id<'users'>`
- `storageIdZodSchema` — branded `Id<'_storage'>`
- `convexPaginationQueryZodSchema` — `{ cursor, limit, sortOrder, search }` for paginated queries
- `sortOrderZodSchema` — `'asc' | 'desc'` with default `'desc'`

Always cast `Id<'table'>` — never use plain `z.string()` for document IDs.

## Query Performance

- **Always** use `.withIndex()` for filtered queries — never full table scans
- `.unique()` for single-result lookups where duplicates are impossible
- `.first()` when the record may not exist (returns `null`, not error)
- `.collect()` for small bounded lists only
- Paginated queries: use `.paginate()` — never load unbounded collections

```ts
// ✅
const user = await ctx.db
  .query('users')
  .withIndex('by_clerkId', (q) => q.eq('clerkId', clerkId))
  .unique();

// ❌ Never
const users = await ctx.db.query('users').collect();
const user = users.find(u => u.email === email);
```

## File Structure

One domain per file. Never create a monolithic `functions.ts`.

```
convex/functions/<domain>/
  index.ts           ← public queries/mutations/actions (exported to client)
  <domain>.utils.ts  ← shared helpers for this domain
  internal.ts        ← internalQuery / internalMutation / internalAction
```

## HTTP Actions

`convex/http.ts` is a **route registry only** — no handler logic inline.

```
convex/
  http.ts                                  ← routes only; import from httpActions/
  httpActions/
    utils.httpActions.ts                   ← CORS headers + JSON response builders
    index.ts                               ← aggregates all handler groups
    <domain>/<domain>.httpActions.ts       ← per-domain httpAction handlers
```

Response helpers for HTTP actions (use instead of bare `new Response`):

```ts
import {
  successJsonResponseForHttpAction,
  errorJsonResponseForHttpAction,
  CORS_HEADERS,
} from '../utils.httpActions';

// Success
return successJsonResponseForHttpAction(200, 'Done.', data);

// Error
return errorJsonResponseForHttpAction(400, 'Invalid input.');

// CORS preflight (OPTIONS)
return new Response(null, { status: 204, headers: CORS_HEADERS });
```

Always register an OPTIONS route for every path that receives cross-origin POST/PATCH/DELETE requests.

Adding a new handler group:
1. Create `convex/httpActions/<domain>/<domain>.httpActions.ts`
2. Export an object of `httpAction(...)` handlers
3. Import and add to `httpActionsHandlers` in `convex/httpActions/index.ts`
4. Register routes in `convex/http.ts`

## Generated Files

**Never write to `convex/_generated/`** — auto-generated by Convex CLI.
Run `npx convex dev` after any schema changes to regenerate types.

Check for `convex/_generated/ai/guidelines.md` before writing any Convex code — if it exists, read it first as it contains project-specific Convex API rules.
