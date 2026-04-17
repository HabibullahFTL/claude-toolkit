# Convex HTTP Actions

Apply sections in order. "Always" sections run unconditionally. "If wants_clerk=true" sections run only when Clerk is being set up.

---

## [Always] `convex/httpActions/utils.httpActions.ts`

```ts
export const CORS_HEADERS = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, PATCH, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
};

// CORS headers are merged into every JSON response — not just OPTIONS preflight.
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

## [Always] `convex/httpActions/index.ts`

```ts
// Import handler groups and re-export them here.
// Example:
//   import { authHttpActions } from './auth/auth.httpActions';
//   export const httpActionsHandlers = { auth: authHttpActions };

export const httpActionsHandlers = {};
```

---

## [Always] `convex/http.ts`

```ts
import { httpRouter } from 'convex/server';
import { httpActionsHandlers } from './httpActions';
import { CORS_HEADERS } from './httpActions/utils.httpActions';
import { httpAction } from './_generated/server';

const http = httpRouter();

// ==================== [CORS preflight] =================
// Register OPTIONS for every path that receives cross-origin POST/PATCH/DELETE requests.
// Example:
// const PREFLIGHT_PATHS = ['/api/auth/login'];
// for (const path of PREFLIGHT_PATHS) {
//   http.route({
//     path,
//     method: 'OPTIONS',
//     handler: httpAction(
//       async () => new Response(null, { status: 204, headers: CORS_HEADERS }),
//     ),
//   });
// }

// Add routes here following this pattern:
// http.route({ path: '/api/...', method: 'POST', handler: httpActionsHandlers.<group>.<handler> });

export default http;
```

---

## [If wants_clerk=true] `convex/httpActions/clerk/clerk.httpActions.ts`

```ts
import { httpAction } from '../../../_generated/server';
import { Webhook } from 'svix';
import { internal } from '../../../_generated/api';

export const clerkHttpActions = {
  webhook: httpAction(async (ctx, request) => {
    const webhookSecret = process.env.CLERK_WEBHOOK_SECRET;
    if (!webhookSecret) {
      return new Response('CLERK_WEBHOOK_SECRET is not set', { status: 500 });
    }

    const svixId = request.headers.get('svix-id');
    const svixTimestamp = request.headers.get('svix-timestamp');
    const svixSignature = request.headers.get('svix-signature');

    if (!svixId || !svixTimestamp || !svixSignature) {
      return new Response('Missing svix signature headers', { status: 400 });
    }

    const body = await request.text();

    let event: { type: string; data: Record<string, unknown> };
    try {
      const wh = new Webhook(webhookSecret);
      event = wh.verify(body, {
        'svix-id': svixId,
        'svix-timestamp': svixTimestamp,
        'svix-signature': svixSignature,
      }) as typeof event;
    } catch {
      return new Response('Invalid webhook signature', { status: 400 });
    }

    if (event.type === 'user.created' || event.type === 'user.updated') {
      const data = event.data as {
        id: string;
        email_addresses: { email_address: string }[];
        first_name?: string | null;
        last_name?: string | null;
      };
      const clerkId = data.id;
      const email = data.email_addresses[0]?.email_address;
      if (!email) {
        return new Response('No email address in event payload', { status: 400 });
      }
      const name =
        [data.first_name, data.last_name].filter(Boolean).join(' ') || undefined;
      await ctx.runMutation(
        internal.functions.users.internal.upsertUserFromClerk,
        { clerkId, email, name },
      );
    }

    return new Response(null, { status: 200 });
  }),
};
```

---

## [If wants_clerk=true] Append to `convex/functions/users/internal.ts`

Read the existing file. Add `internalMutation` to the import from `../../_generated/server` if not already present. Append below `userByClerkIdInternal`:

```ts
// [INTERNAL-FN]: Upsert user from Clerk webhook (user.created / user.updated)
// Finds by clerkId (stable) — also syncs email on user.updated so the DB stays current.
export const upsertUserFromClerk = internalMutation({
  args: {
    clerkId: v.string(),
    email: v.string(),
    name: v.optional(v.string()),
  },
  handler: async (ctx, { clerkId, email, name }) => {
    const existing = await ctx.db
      .query('users')
      .withIndex('by_clerkId', (q) => q.eq('clerkId', clerkId))
      .first();

    if (existing) {
      await ctx.db.patch(existing._id, {
        email, // keep email in sync when user.updated fires
        ...(name !== undefined ? { name } : {}),
      });
    } else {
      await ctx.db.insert('users', {
        clerkId,
        email,
        ...(name ? { name } : {}),
        // ADAPT: add required schema fields here (see SKILL.md Step 5.8 adaptation notes)
      });
    }
  },
});
```

> `v` is already imported from `convex/values` in this file.

---

## [If wants_clerk=true] Patch `convex/httpActions/index.ts`

Two targeted edits to the file just created above:

1. Add import at the top:
   ```ts
   import { clerkHttpActions } from './clerk/clerk.httpActions';
   ```

2. Replace the empty object:
   ```ts
   // before
   export const httpActionsHandlers = {};
   // after
   export const httpActionsHandlers = {
     clerk: clerkHttpActions,
   };
   ```

---

## [If wants_clerk=true] Patch `convex/http.ts`

Add the Clerk webhook route immediately before `export default http;`:

```ts
// ==================== [Clerk Webhooks] =================
http.route({
  path: '/api/webhooks/clerk',
  method: 'POST',
  handler: httpActionsHandlers.clerk.webhook,
});

```

---

## [If wants_file_storage=true] `convex/httpActions/convexStorage/image.ts`

```ts
import { Id } from '../../_generated/dataModel';
import { httpAction } from '../../_generated/server';

export const convexStorageHttpActions = {
  getImageContent: httpAction(async (ctx, request) => {
    const url = new URL(request.url);
    const storageId = url.searchParams.get('storageId');

    if (!storageId) {
      return new Response('Missing storageId parameter', { status: 400 });
    }

    const blob = await ctx.storage.get(storageId as Id<'_storage'>);
    if (!blob) {
      return new Response('Image not found', { status: 404 });
    }

    return new Response(blob, {
      headers: {
        'Content-Type': blob.type,
        'Cache-Control': 'public, max-age=31536000',
      },
    });
  }),
};
```

---

## [If wants_file_storage=true] Patch `convex/httpActions/index.ts`

1. Add import at the top:
   ```ts
   import { convexStorageHttpActions } from './convexStorage/image';
   ```

2. Add `convexStorage` to the handlers object (merge with any existing entries):
   ```ts
   export const httpActionsHandlers = {
     // ...existing entries...
     convexStorage: convexStorageHttpActions,
   };
   ```

---

## [If wants_file_storage=true] Patch `convex/http.ts`

Add the storage route immediately before `export default http;`:

```ts
// =================== [Convex Storage] =================
http.route({
  path: '/convex/storage/get-image',
  method: 'GET',
  handler: httpActionsHandlers.convexStorage.getImageContent,
});

```

---

## Adding a new handler group later (reference — do NOT create during setup)

1. Create `convex/httpActions/<domain>/<domain>.httpActions.ts` — export an object of `httpAction(...)` handlers
2. Import and merge into `httpActionsHandlers` in `convex/httpActions/index.ts`
3. Register routes in `convex/http.ts`
4. If cross-origin POST/PATCH/DELETE, add the path to `PREFLIGHT_PATHS`
