---
description: Clerk auth patterns — server vs client imports, auth() vs currentUser(), getCurrentUser in Convex, middleware, env vars
globs: middleware.ts,app/**/*.ts,app/**/*.tsx,hooks/**/*.ts,convex/functions/**/*.ts
---

# Clerk Rules

## auth() vs currentUser()

| Function | When to use | Cost |
|---|---|---|
| `auth()` | Need only `userId` to check auth or verify identity | Fast — no external call |
| `currentUser()` | Need full Clerk profile (name, email, avatar) directly | Slow — external network call |

**Always prefer the Convex `users` table** over `currentUser()` for profile data. The data is synced via webhook, faster, and works offline.

```ts
// ✅ Lightweight auth check
const { userId } = await auth();
if (!userId) redirect('/sign-in');

// ✅ Full profile — only when Clerk data is explicitly needed
const user = await currentUser();

// ✅ User profile in components — always prefer this
import useConvexQuery from '@/hooks/convex/use-convex-query';
const { data: user } = useConvexQuery(api.functions.users.index.getCurrentUser, {});
```

## Server vs Client Imports

**Server Components / Route Handlers / Convex functions:**
```ts
import { auth, currentUser } from '@clerk/nextjs/server';
const { userId } = await auth();
```

**Client Components:**
```tsx
'use client';
import { useUser, useAuth, useClerk } from '@clerk/nextjs';
const { user, isLoaded } = useUser();
const { userId, isSignedIn } = useAuth();
```

Never import server-only Clerk helpers in `'use client'` files.

## getCurrentUser in Convex

`convexMiddleware` automatically calls `getCurrentUser` before every authenticated function. You receive `currentUser` as the **third argument** to the handler — never call `getCurrentUser` again inside the handler.

`getCurrentUser` queries the `users` table by **email** (from `ctx.auth.getUserIdentity()`). The `users` table must have a `by_email` index.

```ts
// ✅ currentUser is injected by convexMiddleware
handler: async (ctx, args, currentUser) => {
  // currentUser is already Doc<'users'> — use it directly
  const items = await ctx.db
    .query('items')
    .withIndex('by_owner', (q) => q.eq('ownerId', currentUser._id))
    .collect();
}

// ❌ Never re-fetch inside handler
handler: async (ctx, args, currentUser) => {
  const user = await getCurrentUser(ctx); // redundant
}
```

## Middleware

```ts
// ✅ Always use clerkMiddleware (NOT the deprecated authMiddleware)
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher([
  '/',
  '/sign-in(.*)',
  '/sign-up(.*)',
  '/api/webhooks(.*)',
]);

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) await auth.protect();
});
```

## User Sync Strategy

User data syncs from Clerk to Convex via webhook (`/api/webhooks/clerk` → `convex/http.ts`). Never manually sync in components, mutations, or Server Components.

Webhook events to subscribe to: `user.created`, `user.updated` (optionally `user.deleted`).

## Required Environment Variables

Add to `.env.local`:
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>
NEXT_PUBLIC_CONVEX_URL=https://...          # auto-written by npx convex dev
CLERK_WEBHOOK_SECRET=whsec_...             # from Clerk dashboard → Webhooks
```

Also add to Convex dashboard → Settings → Environment Variables:
```
CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>
```
