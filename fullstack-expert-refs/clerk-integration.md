# Clerk Integration — Full Implementations

<!--
  WHEN TO READ THIS FILE:
  - Setting up Clerk for the first time (Scenario A bootstrap).
  - Creating middleware.ts, Providers, getCurrentUser, or userByEmailInternal.
  - User asks to change auth strategy or add a new auth scenario.
  DO NOT read for building features — the usage snippets in the main prompt are sufficient.
  Always check the official Clerk docs (https://clerk.com/docs) for the latest API.
-->

---

## `middleware.ts` — at project root

```ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher([
  '/',
  '/sign-in(.*)',
  '/sign-up(.*)',
  '/api/webhooks(.*)',
]);

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: [
    '/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)',
    '/(api|trpc)(.*)',
  ],
};
```

---

## `app/providers.tsx` — root provider (Convex + Clerk bridge)

`ConvexProviderWithClerk` automatically attaches the Clerk JWT to every Convex request.
`NEXT_PUBLIC_CONVEX_URL` is written to `.env.local` automatically by `npx convex dev`.

```tsx
'use client';
import { ClerkProvider, useAuth } from '@clerk/nextjs';
import { ConvexProviderWithClerk } from 'convex/react-clerk';
import { ConvexReactClient } from 'convex/react';

// Created at module scope — not inside the component — to avoid re-creating on every render
const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <ConvexProviderWithClerk client={convex} useAuth={useAuth}>
        {children}
      </ConvexProviderWithClerk>
    </ClerkProvider>
  );
}
```

---

## Server components / Route handlers — Clerk usage

```ts
import { auth, currentUser } from '@clerk/nextjs/server';

// Get userId only (lightweight — prefer this when you only need to check auth)
const { userId } = await auth();

// Get full user object (heavier — only when you need profile data)
const user = await currentUser();
```

---

## Client components — Clerk usage

```tsx
'use client';
import { useUser, useAuth, useClerk } from '@clerk/nextjs';

const { user, isLoaded } = useUser();
const { userId, isSignedIn } = useAuth();
```

---

## `convex/functions/users/users.utils.ts` — `getCurrentUser`

### Auth & Identity Strategy

Inspect the active project's auth setup before implementing:

- **Scenario 1 — Clerk with Email / Google (standard):**
  Query users by email as the primary unique identifier. Extract from `ctx.auth.getUserIdentity()`.
  Requires a `by_email` index on the `users` table. Ensure `authUser.email` is populated before querying.

- **Scenario 2 — Clerk with Phone / SMS:**
  Fall back to querying by Clerk ID (`authUser.subject`) or phone number.
  Do NOT attempt to query by email in these environments.

```ts
import { internal } from '../../_generated/api';
import { Doc, Id } from '../../_generated/dataModel';
import { ActionCtx, MutationCtx, QueryCtx } from '../../_generated/server';
import { getConvexImageURL } from '../../utils/common';

// Get the currently authenticated user with resolved image URL.
// Called by convexMiddleware on every authenticated function invocation.
export const getCurrentUser = async (
  ctx: QueryCtx | MutationCtx | ActionCtx,
): Promise<Doc<'users'> | undefined> => {
  const authUser = await ctx.auth.getUserIdentity();
  const email = authUser?.email;

  if (!email) {
    return undefined; // No user logged in
  }

  // Fetch user data from DB using email index (runQuery is available in all ctx types)
  const userData = await ctx.runQuery(
    internal.functions.users.internal.userByEmailInternal,
    { email },
  );

  return userData
    ? {
        ...userData,
        // Prefer user-uploaded image; fall back to auth provider's picture URL
        image: userData?.image || authUser?.pictureUrl || '',
      }
    : undefined;
};
```

---

## `convex/functions/users/internal.ts` — `userByEmailInternal`

```ts
import { v } from 'convex/values';
import { Id } from '../../_generated/dataModel';
import { internalQuery } from '../../_generated/server';
import { getConvexImageURL } from '../../utils/common';

// Internal query — not callable from the client, only from other Convex functions
export const userByEmailInternal = internalQuery({
  args: {
    email: v.string(),
  },
  handler: async (ctx, { email }) => {
    const userData = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', email))
      .first();

    if (!userData) {
      return undefined;
    }

    // ⚠️ Adapt to your schema: imageId may be named differently or may not exist
    const imageURL = getConvexImageURL(userData.imageId as Id<'_storage'>);

    return {
      ...userData,
      image: imageURL || '',
    };
  },
});
```

> **Adapt to your schema.** The index name (`by_email`) and field names (`imageId`, etc.)
> must match what your `schema.ts` defines for the `users` table.
