# Convex Middleware — Full Implementations

<!--
  WHEN TO READ THIS FILE:
  - Creating convexMiddleware.ts or convexPublicMiddleware.ts for the first time (Scenario A bootstrap).
  - User explicitly asks to modify or extend middleware logic.
  - Creating a new middleware variant with different auth rules.
  DO NOT read for normal feature development — the usage snippets in the main prompt are sufficient.
-->

---

## `convex/utils/middlewares/convexMiddleware.ts`

Authenticated middleware. Pipeline: Zod validation → auth check → handler.

```ts
import { PropertyValidators } from 'convex/values';
import { z } from 'zod';
import { ActionCtx, MutationCtx, QueryCtx } from '../../_generated/server';
import { getCurrentUser } from '../../functions/users/users.utils';
import { IUser } from '../../types/convex-types';
import { generateConvexErrorResponse, handleConvexZodError } from '../common';
import HttpStatusCodes from '../httpStatusCode';

type IMiddlewareCurrentUser = IUser;

const notAuthorized = (isLoggedIn = false) =>
  generateConvexErrorResponse(
    isLoggedIn ? HttpStatusCodes.FORBIDDEN : HttpStatusCodes.UNAUTHORIZED,
    isLoggedIn
      ? 'You are not authorized to perform this action.'
      : 'Authentication required. Please log in to continue.',
  );

export const convexMiddleware = <
  IArgs extends PropertyValidators,
  ISchema extends z.ZodTypeAny,
  ICtx extends QueryCtx | MutationCtx | ActionCtx,
  IReturn,
>({
  args,
  zodSchema,
  handler,
}: {
  // OPTIONAL RBAC (Role-Based Access Control)
  // permissionKey: 'read_users',      // Unique permission string to check against user's permission list. Implement only if needed.
  // allowedRoles: ['admin'],          // Roles authorized to access this function (e.g., 'admin' | 'user' | 'developer').
  // Note: To use these, ensure 'convexMiddleware' (or similar withMiddleware) is configured to validate these keys against the user's identity.
  args: IArgs;
  zodSchema: ISchema;
  handler: (
    ctx: ICtx,
    arg: z.infer<ISchema>,
    currentUser: IMiddlewareCurrentUser,
  ) => Promise<IReturn>;
}) => {
  return {
    args,
    handler: async (
      ctx: ICtx,
      rawArgs: unknown,
    ): Promise<IReturn | ReturnType<typeof generateConvexErrorResponse>> => {
      // ✅ Step 1: Validate input
      const parsed = zodSchema.safeParse(rawArgs);
      if (!parsed.success) return handleConvexZodError(parsed.error);

      // ✅ Step 2: Validate current user
      const user = await getCurrentUser(ctx);
      // ⚠️ Adapt this check to match your actual `users` table schema.
      // `status` and `isDeleting` are project-specific fields — remove or replace them
      // if your schema uses different field names or has no soft-delete pattern.
      if (!user?._id || user?.status !== 'active' || user?.isDeleting) {
        return notAuthorized();
      }

      // ✅ Step 3: Run business logic
      const currentUser: IMiddlewareCurrentUser = { ...user };
      return handler(ctx, parsed.data, currentUser);
    },
  };
};
```

---

## `convex/utils/middlewares/convexPublicMiddleware.ts`

Public middleware. Skips auth entirely. Validates with Zod if a schema is provided; passes raw args otherwise.

```ts
import { PropertyValidators } from 'convex/values';
import { z } from 'zod';
import { ActionCtx, MutationCtx, QueryCtx } from '../../_generated/server';
import { generateConvexErrorResponse, handleConvexZodError } from '../common';

export const convexPublicMiddleware = <
  IArgs extends PropertyValidators,
  ISchema extends z.ZodTypeAny | undefined,
  ICtx extends QueryCtx | MutationCtx | ActionCtx,
  IReturn,
>(config: {
  args: IArgs;
  zodSchema: ISchema;
  handler: (
    ctx: ICtx,
    args: ISchema extends z.ZodTypeAny ? z.infer<ISchema> : object,
  ) => IReturn | Promise<IReturn>;
}) => {
  const { args: argsFields, zodSchema, handler } = config;
  return {
    args: argsFields,
    handler: async (
      ctx: ICtx,
      args: unknown,
    ): Promise<IReturn | ReturnType<typeof generateConvexErrorResponse>> => {
      // Step 1: Zod validation if schema is provided
      if (zodSchema) {
        const result = zodSchema.safeParse(args);
        if (!result.success) return handleConvexZodError(result.error);
        return handler(
          ctx,
          result.data as ISchema extends z.ZodTypeAny
            ? z.infer<ISchema>
            : object,
        );
      }

      // Step 2: No schema — pass args as-is
      return handler(
        ctx,
        args as ISchema extends z.ZodTypeAny ? z.infer<ISchema> : object,
      );
    },
  };
};
```

---

## Creating a New Middleware Variant

When a project needs different auth rules (e.g. admin-only, subscription check), create a new file:

```
convex/utils/middlewares/
  convexMiddleware.ts          ← standard authenticated
  convexPublicMiddleware.ts    ← unauthenticated
  convexAdminMiddleware.ts     ← new variant: admin-only, subscription check, etc.
```

Reuse `notAuthorized`, `handleConvexZodError`, and `getCurrentUser`. Only change the Step 2 auth logic.
