---
name: setup-next-convex-clerk
description: Scan a Next.js codebase, ask smart contextual questions, then scaffold the full Convex + Clerk boilerplate (middleware, schema, hooks, providers, utils, types) and write stack-specific rules into CLAUDE.md and .claude/rules/**. Works for both fresh Next.js apps and partially-configured existing projects.
tools: Glob, Grep, Read, Bash, Write, Edit, AskUserQuestion, Agent, TaskCreate, TaskUpdate
---

You are setting up a Next.js project with Convex and Clerk. Follow every phase precisely — do not skip or reorder them. Do not write any files until Phase 3.

---

## Phase 0 — Scan the Codebase

Run these **in parallel**:

1. Read `package.json` — extract:
   - `dependencies` and `devDependencies` keys
   - Detect: `next`, `@clerk/nextjs`, `convex`, `tailwindcss`, `zod`, `react-hook-form`, `http-status`, `svix`, `@hookform/resolvers`, `date-fns`, and any `@radix-ui/*` or `@shadcn/ui` packages

2. Check file/directory existence using Glob or Bash `ls`:
   - `convex/` directory
   - `convex/schema.ts`
   - `convex/utils/` directory
   - `convex/functions/` directory
   - `convex/validations/common.ts`
   - `convex/types/convex-types.ts`
   - `convex/utils/middlewares/convexMiddleware.ts`
   - `convex/utils/middlewares/convexPublicMiddleware.ts`
   - `middleware.ts` (project root)
   - `app/providers.tsx` or `app/providers.js`
   - `app/layout.tsx` or `app/layout.js`
   - `hooks/convex/` directory
   - `CLAUDE.md`
   - `.claude/rules/` directory

3. If `app/layout.tsx` exists, read it — note how providers are currently wired.
4. If `convex/schema.ts` exists, read it — note existing tables and fields.
5. Check for `next.config.ts`, `next.config.js`, or `next.config.mjs` — note App Router vs Pages Router.

### Build State Model

After scanning, build this internal state (use it throughout):

```
has_nextjs:              <boolean>  — "next" in package.json
has_clerk:               <boolean>  — "@clerk/nextjs" in package.json
has_convex:              <boolean>  — "convex" in package.json AND convex/ dir exists
has_shadcn:              <boolean>  — any @radix-ui/* package present
has_tailwind:            <boolean>  — "tailwindcss" in package.json
has_http_status:         <boolean>  — "http-status" in package.json
has_zod:                 <boolean>  — "zod" in package.json
has_rhf:                 <boolean>  — "react-hook-form" in package.json
has_middleware:          <boolean>  — middleware.ts exists at root
has_providers:           <boolean>  — app/providers.tsx exists
has_convex_boilerplate:  <boolean>  — convex/utils/middlewares/convexMiddleware.ts exists
has_convex_schema:       <boolean>  — convex/schema.ts exists
has_hooks:               <boolean>  — hooks/convex/ directory exists
has_claude_md:           <boolean>  — CLAUDE.md exists
has_rules_dir:           <boolean>  — .claude/rules/ directory exists
```

### Classify Scenario

- **Scenario A** — Fresh project: has_nextjs is true, has_clerk and has_convex are both false, has_convex_boilerplate is false.
- **Scenario B** — Partial setup: has_nextjs is true, at least one of has_clerk or has_convex is true, but has_convex_boilerplate is false.
- **Scenario C** — Boilerplate already present: has_convex_boilerplate is true. In this case jump directly to Phase 2 (Batch 2 only) then proceed to create only the missing pieces.

---

## Phase 1 — Gather User Input

Use the **AskUserQuestion** tool. Batch related questions together — never ask one at a time.

### Batch 1 — Stack choices (skip if Scenario C)

Build a single question with only the relevant items based on the state model. Include only items that are NOT already installed/present.

Template:
```
I scanned your project and found: [brief summary — e.g. "Next.js installed, Clerk and Convex not yet set up"].

Please answer the following to continue:

[Include only applicable lines below:]

1. 🔐 Authentication — Will you use **Clerk** for auth? (yes / no)
2. 🗄️  Backend/Database — Will you use **Convex** as your real-time backend? (yes / no)
3. 🎨 UI Components — Will you use **ShadCN/UI** for components? (yes / no)
4. 📝 Brief app description — What does this app do? (one line — helps design schema and rules)
```

Record answers as: `wants_clerk`, `wants_convex`, `wants_shadcn`, `app_description`.

If has_clerk is already true, set `wants_clerk = true`.
If has_convex is already true, set `wants_convex = true`.

### Batch 2 — Schema & configuration (only if wants_convex is true)

Ask in a single AskUserQuestion call:

```
A few more details to set up your Convex schema:

1. 👤 Users table extra fields — Beyond the defaults (email, name, imageId), what extra fields do your users need?
   Examples: role ("admin" | "user"), plan ("free" | "pro"), bio, onboardingComplete (boolean)
   → Type them out or say "none" for defaults only.

2. 📦 Additional tables — What other data tables do you know you'll need?
   Example: posts (title, content, status: "published" | "draft"), products (name, price, stock)
   → Describe each or say "none".

3. 🔓 Public routes — Which routes should be accessible without login?
   Defaults already included: /, /sign-in, /sign-up
   → List any extras (e.g. /about, /pricing, /blog) or say "none".
```

Record answers as: `extra_user_fields`, `additional_tables`, `extra_public_routes`.

---

## Phase 2 — Delegate to fullstack-expert Agent

Spawn the **fullstack-expert** agent with this exact brief, substituting all `[PLACEHOLDER]` values with what you collected in Phases 0 and 1.

> **Important:** The fullstack-expert agent has complete knowledge of all file contents and patterns. Do NOT try to write the files yourself — delegate fully. Your job is to pass an accurate, complete brief.

Brief template:

---
```
You are being invoked by the setup-next-convex-clerk skill to scaffold a Next.js + Convex + Clerk project. Follow every instruction below precisely.

## Detected Project State

- Next.js installed: [has_nextjs]
- Clerk installed: [has_clerk] — user wants Clerk: [wants_clerk]
- Convex installed: [has_convex] — user wants Convex: [wants_convex]
- ShadCN/UI present: [has_shadcn] — user wants ShadCN: [wants_shadcn]
- Tailwind CSS: [has_tailwind]
- Zod: [has_zod]
- React Hook Form: [has_rhf]
- http-status package: [has_http_status]
- middleware.ts: [has_middleware]
- app/providers.tsx: [has_providers]
- Convex boilerplate (convexMiddleware): [has_convex_boilerplate]
- convex/schema.ts: [has_convex_schema]
- hooks/convex/: [has_hooks]
- CLAUDE.md: [has_claude_md]
- .claude/rules/: [has_rules_dir]

## Scenario

[Scenario A / B / C — with brief explanation of what this means for what needs to be done]

## User's Answers

- App description: [app_description]
- Extra user fields: [extra_user_fields]
- Additional tables: [additional_tables]
- Extra public routes: [extra_public_routes]

## What to Do

### Step 1 — Install packages

Only install packages that are not already present in package.json.

Detect the package manager first:
- pnpm-lock.yaml → pnpm add
- yarn.lock → yarn add
- Otherwise → npm install

Install as needed (combine into as few commands as possible):
- Clerk (if wants_clerk and not has_clerk): @clerk/nextjs
- Convex (if wants_convex and not has_convex): convex
- http-status (if wants_convex and not has_http_status): http-status
- zod (if wants_convex and not has_zod): zod
- ShadCN: run `npx shadcn@latest init` ONLY if wants_shadcn and not has_shadcn — note to user if not in interactive terminal
- React Hook Form + resolver (if not already present): react-hook-form @hookform/resolvers
- date-fns (if not already present): date-fns
- svix (if wants_clerk and webhook setup needed): svix

### Step 2 — Scaffold boilerplate files

**Follow the bootstrap order from your system prompt exactly** (Scenario A order). Only create files that do not already exist. For existing files, read them first and patch only what is missing.

Bootstrap order:
1. `convex/utils/httpStatusCode.ts`
2. `convex/types/common.ts`
3. `convex/constants/pagination.ts`
4. `convex/validations/common.ts`
5. `convex/types/convex-types.ts`
6. `convex/utils/common.ts`
7. `convex/utils/middlewares/convexMiddleware.ts`
8. `convex/utils/middlewares/convexPublicMiddleware.ts`
9. `middleware.ts` (Clerk — project root) with public routes: /, /sign-in, /sign-up, /api/webhooks(.*)[EXTRA_PUBLIC_ROUTES]
10. `app/providers.tsx` (ConvexProviderWithClerk)
11. Update `app/layout.tsx` — wrap {children} with <Providers>. Read the file first, patch minimally.
12. `convex/functions/users/users.utils.ts` (getCurrentUser)
13. `convex/functions/users/internal.ts` (userByEmailInternal)
14. `hooks/convex/use-convex-query.ts`
15. `hooks/convex/use-convex-mutation.ts`
16. `hooks/convex/use-convex-action.ts`
17. `hooks/convex/use-convex-paginated-query.ts`

### Step 3 — Create convex/schema.ts

[Only if not has_convex_schema]

Create `convex/schema.ts` with:

**users table** — always include these base fields:
- `email: v.string()`
- `name: v.optional(v.string())`
- `imageId: v.optional(v.id('_storage'))`
- Indexes: `.index('by_email', ['email'])`
- Add user-specified extra fields: [extra_user_fields — map to correct v.* validators]
  - string fields → v.string()
  - optional strings → v.optional(v.string())
  - boolean → v.boolean()
  - union/enum (e.g. 'admin' | 'user') → v.union(v.literal('admin'), v.literal('user'))
  - number → v.number()

**Additional tables** — for each table in [additional_tables]:
- Add sensible fields based on the description
- Add `_creationTime` is auto-added by Convex — do NOT add it manually
- Add at least one lookup index per table

[If has_convex_schema: read the existing schema and ADD the new tables/fields without removing anything existing]

### Step 4 — Write rules files

Create `.claude/rules/` directory if not exists.

#### `.claude/rules/convex.md`

```markdown
---
description: Convex function authoring rules — middleware wrappers, response helpers, comment tags, schema patterns
globs: convex/**/*.ts,convex/**/*.tsx
---

# Convex Rules

## Middleware — Required on Every Function

Never write a bare `query`, `mutation`, or `action`. Always wrap:

| Wrapper | Use when |
|---|---|
| `convexMiddleware` | Requires authenticated user |
| `convexPublicMiddleware` | Public / no auth required |

Import from `../utils/middlewares/convexMiddleware` or `convexPublicMiddleware`.

## Comment Tags

Place directly above every exported Convex function:

| Tag | Use when |
|---|---|
| `// [USER-FN]: description` | Authenticated, regular user |
| `// [ADMIN-FN]: description` | Admin role required |
| `// [PUBLIC-FN]: description` | No auth required |
| `// [INTERNAL-FN]: description` | Convex internal functions |

## Response Helpers — Never Throw

**Never use `throw new Error()` or `ConvexError`.** Always return:

```ts
// Success
return generateConvexSuccessResponse(HttpStatusCodes.OK, 'Message.', data);
return generateConvexSuccessResponse(HttpStatusCodes.CREATED, 'Created.', data);

// Error
return generateConvexErrorResponse(HttpStatusCodes.BAD_REQUEST, 'Message.');
return generateConvexErrorResponse(HttpStatusCodes.NOT_FOUND, 'Not found.');
```

Import from `../../utils/common`. Use `HttpStatusCodes` from `../../utils/httpStatusCode` — never hardcode numbers.

## Zod Validation

Every `convexMiddleware` / `convexPublicMiddleware` call must include a `zodSchema`. Reuse schemas from `convex/validations/common.ts` where applicable:
- `emailZodSchema` — validated email
- `userIdZodSchema` — branded `Id<'users'>`
- `convexPaginationQueryZodSchema` — paginated query args
- `sortOrderZodSchema` — `'asc' | 'desc'`

## Query Performance

- Always use `.withIndex()` for filtered queries — never full table scans
- Use `.unique()` for single-result lookups; `.first()` when the record may not exist; `.collect()` for lists
- Paginated queries: use `.paginate()` — never load unbounded collections with `.collect()`

## File Structure

One domain per file. Never create a monolithic `functions.ts`.

```
convex/functions/<domain>/
  index.ts          ← exported queries/mutations/actions
  <domain>.utils.ts ← shared helpers for this domain
  internal.ts       ← internalMutation / internalAction
```

## Generated Files

Never write to `convex/_generated/` — these are auto-generated. Run `npx convex dev` after schema changes.
```

#### `.claude/rules/hooks.md`

```markdown
---
description: Custom hook conventions — wrapping Convex hooks, domain hooks, file location
globs: hooks/**/*.ts,hooks/**/*.tsx,app/**/_hooks/**/*.ts
---

# Hook Rules

## Never Use Convex Hooks Directly in Components

**Never** call `useQuery`, `useMutation`, or `useAction` directly in components.
Always use the custom wrapper hooks:

| Hook | Replaces | Returns |
|---|---|---|
| `useConvexQuery(fn, args)` | `useQuery` | `{ data, error, isLoading }` |
| `useConvexMutation(fn, opts?)` | `useMutation` | `{ data, error, isLoading, isSuccess, isError, isSettled, mutate, reset }` |
| `useConvexAction(fn, opts?)` | `useAction` | same shape but `call` instead of `mutate` |
| `useConvexPaginatedQuery(fn, args)` | `usePaginatedQuery` | `{ data, meta, error, isLoading, pagination }` |

## Response Unwrapping

Convex functions return `IConvexResult` envelopes — always check `result.success` before accessing `.data`:

```ts
const { data, error, isLoading } = useConvexQuery(api.functions.items.index.readItems, { sortOrder: 'desc' });
// data is already unwrapped (IConvexResponse.data) — never check .success on the hook return
```

`onSuccess` in mutation/action hooks fires **only** when `result.success === true`.
The callback receives the full envelope — access the payload via `data.data`.

## Domain Hooks

Wrap core hooks in domain-specific hooks to expose clean typed APIs:

```ts
// hooks/use-create-item.ts
export function useCreateItem() {
  return useConvexMutation(api.functions.items.index.createItem, {
    onSuccess: (data) => { /* toast, redirect, etc. */ },
  });
}
```

## File Location

- `hooks/convex/` — core wrapper hooks (use-convex-query, use-convex-mutation, etc.)
- `hooks/` root — reusable domain hooks (use-create-item, use-current-user, etc.)
- `app/<route>/_hooks/` — page-local hooks; never import from other routes
- One hook per file; `use-kebab-case.ts` naming
```

#### `.claude/rules/nextjs.md`

```markdown
---
description: Next.js App Router conventions — Server Components, page structure, data fetching, metadata
globs: app/**/*.ts,app/**/*.tsx,components/**/*.ts,components/**/*.tsx
---

# Next.js App Router Rules

## Server Components by Default

Add `"use client"` **only** when the component uses:
- React hooks (`useState`, `useEffect`, etc.)
- Browser APIs
- Convex real-time hooks (`useConvexQuery`, etc.)
- Event handlers that cannot be Server Actions

Never add `"use client"` to layouts unless absolutely necessary.

## page.tsx — Compose Only

`page.tsx` must be a Server Component. It composes — no business logic, no inline JSX beyond composition.

```tsx
// ✅ Correct
const DashboardPage = async () => {
  const preloaded = await preloadQuery(api.functions.items.index.readItems, { sortOrder: 'desc' });
  return <DashboardView preloadedItems={preloaded} />;
};

// ❌ Wrong — business logic in page
const DashboardPage = async () => {
  const items = await fetchItems(); // no
  return <div>{items.map(...)}</div>; // no
};
```

## SSR Data Fetching — preloadQuery Pattern

Pass server-preloaded data to Client Components via `preloadQuery` / `usePreloadedQuery`:

```tsx
// Server: page.tsx
import { preloadQuery } from 'convex/nextjs';
const preloaded = await preloadQuery(api.functions.items.index.readItems, args);
return <View preloadedItems={preloaded} />;

// Client: _components/view.tsx
'use client';
import { usePreloadedQuery } from 'convex/react';
const result = usePreloadedQuery(preloadedItems);
const items = result?.success ? result.data : [];
```

Always unwrap via `result?.success` — never access `.data` without checking `.success` first.

## Required Per Route

- `loading.tsx` — shown while the page suspends
- `error.tsx` — shown on unhandled errors (must be `'use client'`)
- `generateMetadata` — export for all public-facing pages

## Component Folder Structure

```
app/<route>/
  page.tsx              ← Server Component, compose only
  loading.tsx
  error.tsx
  _components/          ← page-local components (never import elsewhere)
    <feature>/
      index.tsx
      <sub-component>.tsx
  _utils/               ← page-local helpers
  _hooks/               ← page-local hooks
```

`_components/`, `_utils/`, `_hooks/` are private to their route. Never import them from another route.

## Reusable Components

```
components/
  ui/         ← ShadCN generated — never edit directly; wrap to extend
  shared/     ← reusable cross-route components
```

## Assets and Utilities

- `next/image` for all images — never raw `<img>`
- `next/link` for all internal navigation — never `<a href>`
- `next/font` for fonts — never CSS `@import` for fonts
```

#### `.claude/rules/clerk.md`

```markdown
---
description: Clerk authentication rules — server vs client usage, getCurrentUser, middleware
globs: middleware.ts,app/**/*.ts,app/**/*.tsx,hooks/**/*.ts,convex/functions/**/*.ts
---

# Clerk Rules

## auth() vs currentUser()

| Function | Use when | Speed |
|---|---|---|
| `auth()` | Only need `userId` to check auth or pass to Convex | Fast — no external call |
| `currentUser()` | Need full Clerk profile (name, email, avatar) directly | Slow — external fetch |

**Prefer Convex `users` table** over `currentUser()` in most cases — the data is already synced, faster, and offline-tolerant.

## Server vs Client

**Server Components / Route Handlers:**
```ts
import { auth, currentUser } from '@clerk/nextjs/server';
const { userId } = await auth();
```

**Client Components:**
```tsx
'use client';
import { useUser, useAuth } from '@clerk/nextjs';
const { user, isLoaded } = useUser();
```

## getCurrentUser in Convex

`convexMiddleware` automatically calls `getCurrentUser` on every authenticated function — you receive `currentUser` as the third handler argument. Never call it again inside the handler.

`getCurrentUser` queries users by **email** (not Clerk ID). Requires `by_email` index on the `users` table.

## User Sync

User data syncs to Convex via the Clerk webhook at `/api/webhooks/clerk` (handled in `convex/http.ts`). Never manually sync in components or mutations.

## Middleware

```ts
// middleware.ts — always use clerkMiddleware (not deprecated authMiddleware)
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';
const isPublicRoute = createRouteMatcher(['/sign-in(.*)', '/sign-up(.*)', ...]);
export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) await auth.protect();
});
```

## Environment Variables

Required in `.env.local`:
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>
NEXT_PUBLIC_CONVEX_URL=https://...
```

Also set `CLERK_JWT_ISSUER_DOMAIN` in Convex dashboard → Settings → Environment Variables.
```

### Step 5 — Update CLAUDE.md

[Read existing CLAUDE.md if it exists. Append the section below — never duplicate existing content.]

Append:

```markdown
## Stack

- **Next.js** — App Router, Server Components first
- **Convex** — real-time backend (queries, mutations, actions, HTTP endpoints)
- **Clerk** — authentication (clerkMiddleware + ConvexProviderWithClerk)
- **TypeScript** — strict; no `any`, no implicit casts
- **Tailwind CSS** — utility-first; no custom CSS
- **ShadCN/UI** — radix-ui primitives; never base-ui
- **Zod** — validation in Convex middleware and forms
- **React Hook Form** — form state; always pair with zodResolver

## Architecture

All database access goes through Convex functions in `convex/functions/`. Components never call the DB directly.

Every Convex function is wrapped with `convexMiddleware` (auth required) or `convexPublicMiddleware` (public). Never write bare `query`, `mutation`, or `action`.

User data syncs from Clerk to Convex via webhook — read user profile from the Convex `users` table, not from Clerk directly.

`app/providers.tsx` bridges Clerk auth to Convex via `ConvexProviderWithClerk`.

## Key Conventions

- **Never throw in Convex functions** — always return `generateConvexSuccessResponse` / `generateConvexErrorResponse`
- **Never hardcode HTTP status codes** — use `HttpStatusCodes` from `convex/utils/httpStatusCode.ts`
- **Never use `useQuery`/`useMutation`/`useAction` directly** — use custom wrappers from `hooks/convex/`
- **page.tsx is compose-only** — no business logic, no inline JSX
- Comment every Convex export with a tag: `// [USER-FN]:`, `// [PUBLIC-FN]:`, `// [INTERNAL-FN]:`

## Convex Folder Structure

```
convex/
  functions/<domain>/
    index.ts          ← public queries/mutations/actions
    <domain>.utils.ts ← domain helpers
    internal.ts       ← internalMutation / internalAction
  validations/common.ts
  types/convex-types.ts, common.ts
  constants/pagination.ts
  utils/common.ts, httpStatusCode.ts
  utils/middlewares/convexMiddleware.ts, convexPublicMiddleware.ts
  schema.ts
  http.ts
  httpActions/<domain>/<domain>.httpActions.ts
```

## Hook Structure

```
hooks/
  convex/
    use-convex-query.ts
    use-convex-mutation.ts
    use-convex-action.ts
    use-convex-paginated-query.ts
  use-<domain>-<action>.ts    ← domain-specific hooks
```
```

### Step 6 — Final notes

After completing all steps, report what was done. See Phase 3 below for the report format.
```
---

(End of brief template)

---

## Phase 3 — Report

After the agent completes, output a clean summary:

```
✅ Setup complete

SCENARIO: [A / B / C]

INSTALLED:
  • @clerk/nextjs [version]
  • convex [version]
  • http-status [version]
  [list others, or "nothing new" if all were already present]

CREATED:
  • convex/utils/httpStatusCode.ts
  • convex/types/common.ts
  • convex/constants/pagination.ts
  • convex/validations/common.ts
  • convex/types/convex-types.ts
  • convex/utils/common.ts
  • convex/utils/middlewares/convexMiddleware.ts
  • convex/utils/middlewares/convexPublicMiddleware.ts
  • middleware.ts
  • app/providers.tsx
  • convex/functions/users/users.utils.ts
  • convex/functions/users/internal.ts
  • convex/schema.ts  (tables: users[, extra tables])
  • hooks/convex/use-convex-query.ts
  • hooks/convex/use-convex-mutation.ts
  • hooks/convex/use-convex-action.ts
  • hooks/convex/use-convex-paginated-query.ts
  • .claude/rules/convex.md
  • .claude/rules/hooks.md
  • .claude/rules/nextjs.md
  • .claude/rules/clerk.md
  [mark SKIPPED — already existed for any file not created]

UPDATED:
  • app/layout.tsx  (wrapped {children} with <Providers>)
  • CLAUDE.md  (appended stack conventions)

⚠️  NEXT STEPS — Required before the app will work:

1. Run `npx convex dev` — initializes Convex and writes NEXT_PUBLIC_CONVEX_URL to .env.local
2. Create a Clerk app at https://dashboard.clerk.com → copy your API keys
3. Add to `.env.local`:
     NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
     CLERK_SECRET_KEY=sk_...
     CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>
4. In Convex dashboard → Settings → Environment Variables:
     Add: CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>
5. In Clerk dashboard → Webhooks:
     Endpoint: <your-convex-site-url>/api/webhooks/clerk
     Events: user.created, user.updated
     Copy the signing secret → add to .env.local: CLERK_WEBHOOK_SECRET=whsec_...
6. Restart dev server — run `npx convex dev` in one terminal, `npm run dev` in another
```

If any step was skipped or failed, state it with the reason.
