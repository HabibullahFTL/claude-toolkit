# Convex User Functions

Write each file below to the exact path shown.

> **Adaptation note for `internal.ts`:**
> If the user said they do NOT need file storage (`imageId`), remove the `getConvexImageURL`
> call and return `userData` directly without the `image` computed field.

---

## `convex/functions/users/users.utils.ts`

```ts
import { internal } from '../../_generated/api';
import { ActionCtx, MutationCtx, QueryCtx } from '../../_generated/server';
import { IUserWithImage } from '../../types/convex-types';

// Get the currently authenticated user.
// Called by convexMiddleware on every authenticated function invocation.
// Uses authUser.subject (Clerk's stable user ID) — survives email changes and multi-provider auth.
export const getCurrentUser = async (
  ctx: QueryCtx | MutationCtx | ActionCtx,
): Promise<IUserWithImage | undefined> => {
  const authUser = await ctx.auth.getUserIdentity();
  const clerkId = authUser?.subject;

  if (!clerkId) {
    return undefined; // No user logged in
  }

  const userData = await ctx.runQuery(
    internal.functions.users.internal.userByClerkIdInternal,
    { clerkId },
  );

  if (!userData) return undefined;

  return {
    ...userData,
    // Prefer user-uploaded image; fall back to auth provider's picture URL
    image: userData.image || authUser?.pictureUrl || '',
  } satisfies IUserWithImage;
};
```

---

## `convex/functions/users/internal.ts`

```ts
import { v } from 'convex/values';
import { Id } from '../../_generated/dataModel';
import { internalQuery } from '../../_generated/server';
import { getConvexImageURL } from '../../utils/common';

// Internal query — not callable from the client, only from other Convex functions.
// Looks up a user by Clerk's stable subject ID using the by_clerkId index (must exist in schema.ts).
// Using clerkId (not email) ensures lookup survives email changes and multi-provider auth.
export const userByClerkIdInternal = internalQuery({
  args: {
    clerkId: v.string(),
  },
  handler: async (ctx, { clerkId }) => {
    const userData = await ctx.db
      .query('users')
      .withIndex('by_clerkId', (q) => q.eq('clerkId', clerkId))
      .first();

    if (!userData) {
      return undefined;
    }

    // Resolve imageId (Convex storage ID) to a public URL.
    // If the project does not use file storage, remove this and return userData directly.
    const imageURL = getConvexImageURL(userData.imageId as Id<'_storage'>);

    return {
      ...userData,
      image: imageURL || '',
    };
  },
});
```
