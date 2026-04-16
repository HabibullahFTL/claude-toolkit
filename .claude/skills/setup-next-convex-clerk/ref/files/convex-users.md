# Convex User Functions

Write each file below to the exact path shown.

> **Adaptation note for `internal.ts`:**
> If the user said they do NOT need file storage (`imageId`), remove the `getConvexImageURL`
> call and return `userData` directly without the `image` computed field.

---

## `convex/functions/users/users.utils.ts`

```ts
import { internal } from '../../_generated/api';
import { Doc } from '../../_generated/dataModel';
import { ActionCtx, MutationCtx, QueryCtx } from '../../_generated/server';

// Get the currently authenticated user.
// Called by convexMiddleware on every authenticated function invocation.
// Queries users by email (standard Clerk/email auth).
// For phone/SMS auth, replace email lookup with Clerk subject ID lookup.
export const getCurrentUser = async (
  ctx: QueryCtx | MutationCtx | ActionCtx,
): Promise<Doc<'users'> | undefined> => {
  const authUser = await ctx.auth.getUserIdentity();
  const email = authUser?.email;

  if (!email) {
    return undefined; // No user logged in
  }

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

## `convex/functions/users/internal.ts`

```ts
import { v } from 'convex/values';
import { Id } from '../../_generated/dataModel';
import { internalQuery } from '../../_generated/server';
import { getConvexImageURL } from '../../utils/common';

// Internal query — not callable from the client, only from other Convex functions.
// Looks up a user by email using the by_email index (must exist in schema.ts).
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
