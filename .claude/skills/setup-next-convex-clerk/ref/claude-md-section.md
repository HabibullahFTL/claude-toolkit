## Stack

- **Next.js** — App Router, Server Components first; no Pages Router
- **Convex** — real-time backend (queries, mutations, actions, HTTP endpoints)
- **Clerk** — authentication via `clerkMiddleware` + `ConvexProviderWithClerk`
- **TypeScript** — strict mode; no `any`, no implicit casts
- **Tailwind CSS** — utility-first; no custom CSS files
- **ShadCN/UI** — radix-ui primitives only; never base-ui; never edit `components/ui/` directly
- **Zod** — validation in Convex middleware and forms (with React Hook Form)
- **date-fns** — all date formatting and manipulation; never raw `Date` methods in display code

## Architecture

All database access goes through Convex functions in `convex/functions/`. Components never touch the DB directly.

Every Convex function is wrapped with `convexMiddleware` (auth required) or `convexPublicMiddleware` (public). **Never write a bare `query`, `mutation`, or `action`.**

User data syncs from Clerk to Convex via webhook — read user profile from the Convex `users` table, not from Clerk's `currentUser()`.

`app/providers.tsx` bridges Clerk auth to Convex via `ConvexProviderWithClerk`.

## Non-Negotiable Conventions

| Rule | Detail |
|---|---|
| Never throw in Convex functions | Always return `generateConvexSuccessResponse` / `generateConvexErrorResponse` |
| Never hardcode HTTP status codes | Use `HttpStatusCodes` from `convex/utils/httpStatusCode.ts` |
| Never use Convex hooks directly | Always use wrappers from `hooks/convex/` |
| Never call `useQuery`/`useMutation` in components | Use `useConvexQuery`, `useConvexMutation`, `useConvexAction` |
| Comment every Convex export | `// [USER-FN]:`, `// [PUBLIC-FN]:`, `// [INTERNAL-FN]:`, `// [ADMIN-FN]:` |
| `page.tsx` is compose-only | No business logic; no inline JSX beyond composition |

## Convex Folder Structure

```
convex/
  functions/
    users/
      index.ts          ← public queries/mutations
      users.utils.ts    ← getCurrentUser helper
      internal.ts       ← internalQuery (userByClerkIdInternal)
    <domain>/
      index.ts
      <domain>.utils.ts
      internal.ts
  validations/
    common.ts           ← reusable Zod schemas
  types/
    convex-types.ts     ← IConvexResponse, IConvexErrorResponse, IUser, etc.
    common.ts           ← ISortOrder
  constants/
    pagination.ts       ← DEFAULT_PAGE_LIMIT, DEFAULT_SORT_ORDER
  utils/
    common.ts           ← generateConvexSuccessResponse, generateConvexErrorResponse
    httpStatusCode.ts   ← re-exports http-status package
    middlewares/
      convexMiddleware.ts
      convexPublicMiddleware.ts
  schema.ts
  http.ts               ← route registry only; handlers in httpActions/
  httpActions/
    utils.httpActions.ts
    <domain>/<domain>.httpActions.ts
```

## Hook Structure

```
hooks/
  convex/
    use-convex-query.ts
    use-convex-mutation.ts
    use-convex-action.ts
    use-convex-paginated-query.ts
  use-<domain>-<action>.ts    ← domain-specific wrappers
```

## Key Types

- `IConvexResponse<TData, TMeta>` — `{ success: true, statusCode, message, data, meta? }`
- `IConvexErrorResponse` — `{ success: false, statusCode, message, errorSources? }`
- `IConvexResult<TData, TMeta>` — union of both
- `IUser` — `Doc<'users'>` (raw schema row; extends automatically with schema fields)
- `IUserWithImage` — `IUser & { image: string }` — the type of `currentUser` in middleware handlers
- `ISortOrder` — `'asc' | 'desc'`
- `Doc<'tableName'>` / `Id<'tableName'>` — from `convex/_generated/dataModel`
