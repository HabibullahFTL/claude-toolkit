# Convex Middleware Files

Write each file below to the exact path shown.

> **Adaptation required for `convexMiddleware.ts`:**
> The Step 2 user validation check is project-specific. The SKILL.md will tell you which
> variant to use based on the user's schema answers. The three variants are marked below.

---

## `convex/utils/middlewares/convexMiddleware.ts`

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
      // ADAPT THIS CHECK — three variants depending on schema:
      //
      // Variant A — no status or soft-delete fields (default):
      //   if (!user?._id) { return notAuthorized(); }
      //
      // Variant B — schema has a status field (e.g. status: 'active' | 'suspended'):
      //   if (!user?._id || user?.status !== 'active') { return notAuthorized(); }
      //
      // Variant C — schema has status + soft-delete (isDeleting):
      //   if (!user?._id || user?.status !== 'active' || user?.isDeleting) { return notAuthorized(); }
      //
      const user = await getCurrentUser(ctx);
      if (!user?._id) {
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
          result.data as ISchema extends z.ZodTypeAny ? z.infer<ISchema> : object,
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
