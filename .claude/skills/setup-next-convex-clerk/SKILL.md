---
name: setup-next-convex-clerk
description: Scan a Next.js codebase, ask focused questions, then install packages, scaffold the full Convex + Clerk boilerplate (middleware, schema, hooks, providers, utils, types), and write stack-specific rules into CLAUDE.md and .claude/rules/**. No subagent — Claude executes everything directly using ref files stored alongside this skill.
tools: Glob, Grep, Read, Bash, Write, Edit, AskUserQuestion, TaskCreate, TaskUpdate
---

You are setting up a Next.js project with Convex and Clerk. Execute every phase yourself using the tools above. Do not delegate to a subagent.

## Reference Files

This skill stores boilerplate in a `ref/` directory next to this file. The skill base directory is shown in the `Base directory for this skill:` line in the `<command-message>` header of this conversation.

At the appropriate phase, read each ref file using the **Read** tool and write its contents to the user's project verbatim (with adaptations noted per phase). Ref file paths:

```
<base_dir>/ref/files/convex-utils.md           → 6 utility files (httpStatusCode, types, constants, validations, convex-types, common)
<base_dir>/ref/files/convex-middlewares.md  → convexMiddleware + convexPublicMiddleware
<base_dir>/ref/files/convex-users.md        → users.utils.ts + users/internal.ts
<base_dir>/ref/files/convex-http-actions.md → HTTP actions scaffold + optional Clerk webhook handler (conditional sections)
<base_dir>/ref/files/clerk-integration.md   → middleware.ts + app/providers.tsx
<base_dir>/ref/files/hooks.md               → 4 custom Convex hooks
<base_dir>/ref/rules/convex.md              → .claude/rules/convex.md content
<base_dir>/ref/rules/hooks.md               → .claude/rules/hooks.md content
<base_dir>/ref/rules/nextjs.md              → .claude/rules/nextjs.md content
<base_dir>/ref/rules/clerk.md               → .claude/rules/clerk.md content
<base_dir>/ref/claude-md-section.md         → CLAUDE.md section to append
```

---

## Phase 0 — Scan the Codebase

Run these reads **in parallel**:

1. Read `package.json` — extract `dependencies` and `devDependencies`. Look for:
   `next`, `@clerk/nextjs`, `convex`, `tailwindcss`, `zod`, `react-hook-form`,
   `@hookform/resolvers`, `http-status`, `svix`, `date-fns`, any `@radix-ui/*` packages

2. Check existence of these paths (Glob or Bash):
   - `convex/` directory
   - `convex/schema.ts`
   - `convex/utils/middlewares/convexMiddleware.ts`
   - `convex/functions/users/users.utils.ts`
   - `convex/http.ts`
   - `convex/httpActions/clerk/`
   - `middleware.ts`
   - `app/providers.tsx`
   - `app/layout.tsx`
   - `hooks/convex/`
   - `CLAUDE.md`
   - `.claude/rules/`

3. If `app/layout.tsx` exists → read it (to know how providers are wired)
4. If `convex/schema.ts` exists → read it (to know existing tables)

### Build State Model

```
has_clerk:               "@clerk/nextjs" in package.json
has_convex:              "convex" in package.json AND convex/ dir exists
has_shadcn:              any @radix-ui/* package present
has_tailwind:            "tailwindcss" in package.json
has_zod:                 "zod" in package.json
has_rhf:                 "react-hook-form" in package.json
has_http_status:         "http-status" in package.json
has_middleware:          middleware.ts exists at project root
has_providers:           app/providers.tsx exists
has_boilerplate:         convex/utils/middlewares/convexMiddleware.ts exists
has_user_functions:      convex/functions/users/users.utils.ts exists
has_convex_schema:       convex/schema.ts exists
has_http_router:         convex/http.ts exists
has_webhook_handler:     convex/httpActions/clerk/ directory exists
has_hooks:               hooks/convex/ directory exists
has_claude_md:           CLAUDE.md exists
has_rules_dir:           .claude/rules/ directory exists
```

### Classify Scenario

- **Scenario A** — `has_clerk=false` AND `has_convex=false` (fresh Next.js app)
- **Scenario B** — at least one of `has_clerk` or `has_convex` is true, but `has_boilerplate=false`
- **Scenario C** — `has_boilerplate=true` (boilerplate already present; create only what's missing)

---

## Phase 1 — Batch A: Core Stack Questions

Skip any question for a package already detected — auto-set its flag to `true`. Show the scan summary in the first question. Ask each as its own AskUserQuestion call:

**Call 1 — Clerk** (skip if `has_clerk=true`, auto-set `wants_clerk=true`):
- question: `"I've scanned your project. Here's what I found:\n[brief bullet list — e.g. ✅ Next.js 15 | ❌ Clerk | ❌ Convex | ✅ Tailwind]\n\n🔐 Will you use Clerk for authentication?"`
- options: `["Yes", "No"]`

**Call 2 — Convex** (skip if `has_convex=true`, auto-set `wants_convex=true`):
- question: `"🗄️ Will you use Convex as your real-time backend/database?"`
- options: `["Yes", "No"]`

**Call 3 — ShadCN/UI** (skip if `has_shadcn=true`, auto-set `wants_shadcn=true`):
- question: `"🎨 Will you use ShadCN/UI for components?"`
- options: `["Yes", "No"]`

**Call 4 — App description** (always ask):
- question: `"📝 Briefly describe what this app does. (One line — helps design the schema and rules.)"`
- (no options — free text)

Record: `wants_clerk`, `wants_convex`, `wants_shadcn`, `app_description`

**If Scenario C:** Skip Batch A entirely. Set `wants_clerk = has_clerk`, `wants_convex = has_convex`, and `wants_shadcn = has_shadcn` (answers are already known from the scan). If `has_shadcn=false`, still ask Call 3 (ShadCN question) — it cannot be derived from the scan. Then go to Batch B and Batch C.

---

## Phase 2 — Batch B: Clerk Questions

**Only ask if `wants_clerk=true`.** Ask each as its own AskUserQuestion call:

**Call 1 — Public routes** (free text):
- question: `"🔓 Which routes should be accessible without login?\nAlready included: /, /sign-in(.*), /sign-up(.*), /api/webhooks(.*)\nList any extra paths (e.g. /about /pricing /blog), or leave blank for none."`
- (no options — free text)

**Call 2 — Organizations**:
- question: `"🏢 Does your app need Clerk Organizations (multi-tenant support)?\nMost apps don't — select No if unsure."`
- options: `["Yes", "No"]`

**Call 3 — After sign-in URL**:
- question: `"↩️ Where should users land after signing in?"`
- options: `["/dashboard", "/onboarding", "Other (type below)"]`

**Call 4 — After sign-up URL**:
- question: `"↩️ Where should users land after signing up?"`
- options: `["/dashboard", "/onboarding", "Other (type below)"]`

For Calls 3 and 4: if the user selects "Other (type below)", use their typed value as the URL. If they select a listed path, use that value directly. Default to `/dashboard` if nothing is provided.

Record: `extra_public_routes` (list), `wants_orgs` (bool), `after_sign_in_url`, `after_sign_up_url`

---

## Phase 3 — Batch C: Convex Questions

**Only ask if `wants_convex=true`.** Ask each as its own AskUserQuestion call:

**Call 1 — Extra user fields** (free text):
- question: `"👤 Beyond the defaults (email, name, imageId), what extra fields do your users need?\nExamples:\n  role: \"admin\" | \"user\"\n  plan: \"free\" | \"pro\" | \"enterprise\"\n  onboardingComplete: boolean\n  bio: string (optional)\nList them with types, or type \"none\" for defaults only."`
- (no options — free text)

**Call 2 — User status / soft-delete**:
- question: `"🔄 Does your users table need a status or soft-delete field?\n• Status only — adds status: \"active\" | \"suspended\" (gates middleware access)\n• Soft-delete only — adds isDeleting: boolean\n• Both — adds both fields\n• None — no extra fields"`
- options: `["None", "Status only", "Soft-delete only", "Both"]`

Map to `user_status_config`: `None → none`, `Status only → status`, `Soft-delete only → soft-delete`, `Both → both`.

**Call 3 — Additional tables** (free text):
- question: `"🗃️ What other data tables do you know you'll need?\nExample: posts (title, content, authorId, status: \"published\" | \"draft\")\nDescribe each, or type \"none\"."`
- (no options — free text)

**Call 4 — File storage**:
- question: `"🖼️ Does this app need Convex file storage for uploads (avatars, images, etc.)?\nSelecting No removes imageId from the users table."`
- options: `["Yes", "No"]`

Record: `extra_user_fields`, `user_status_config` (`none`/`status`/`soft-delete`/`both`), `additional_tables`, `wants_file_storage`

---

## Phase 4 — Install Packages

Detect the package manager first:

```bash
ls pnpm-lock.yaml yarn.lock package-lock.json 2>/dev/null | head -1
```

- `pnpm-lock.yaml` → `pnpm add`
- `yarn.lock` → `yarn add`
- otherwise → `npm install`

Install only packages NOT already in `package.json`. Combine into as few commands as possible:

| Package                               | When                                          |
| ------------------------------------- | --------------------------------------------- |
| `@clerk/nextjs`                       | `wants_clerk=true` AND NOT `has_clerk`        |
| `convex`                              | `wants_convex=true` AND NOT `has_convex`      |
| `http-status`                         | `wants_convex=true` AND NOT `has_http_status` |
| `zod`                                 | `wants_convex=true` AND NOT `has_zod`         |
| `react-hook-form @hookform/resolvers` | NOT `has_rhf`                                 |
| `date-fns`                            | not already installed                         |
| `svix`                                | `wants_clerk=true` (for webhook verification) |

For ShadCN (`wants_shadcn=true` AND NOT `has_shadcn`): Run `npx shadcn@latest init`. If the terminal is non-interactive, skip and note it in the final report — the user must run it manually.

---

## Phase 5 — Scaffold Files

**Only create files that do not already exist.** For existing files, read them first and patch only what is missing.

Read the skill base directory from the `<command-message>` header. Then read and apply ref files in this exact order:

### Step 5.1 — Convex utilities (if `wants_convex=true` AND NOT `has_boilerplate`)

Read `<base_dir>/ref/files/convex-utils.md`. Write each of the 6 files listed verbatim:

1. `convex/utils/httpStatusCode.ts`
2. `convex/types/common.ts`
3. `convex/constants/pagination.ts`
4. `convex/validations/common.ts`
5. `convex/types/convex-types.ts`
6. `convex/utils/common.ts`

Create all parent directories as needed.

### Step 5.2 — Convex middlewares (if `wants_convex=true` AND NOT `has_boilerplate`)

Read `<base_dir>/ref/files/convex-middlewares.md`. Write both files:

- `convex/utils/middlewares/convexMiddleware.ts`
- `convex/utils/middlewares/convexPublicMiddleware.ts`

**Adapt `convexMiddleware.ts` Step 2** based on `user_status_config`. Replace the comment + check block with the appropriate variant. `notAuthenticated()` returns 401; `notAuthorized()` returns 403:

- `none`:
  ```ts
  const user = await getCurrentUser(ctx);
  if (!user?._id) return notAuthenticated();
  ```
- `status`:
  ```ts
  const user = await getCurrentUser(ctx);
  if (!user?._id) return notAuthenticated();
  if (user.status !== 'active') return notAuthorized();
  ```
- `soft-delete`:
  ```ts
  const user = await getCurrentUser(ctx);
  if (!user?._id) return notAuthenticated();
  if (user.isDeleting) return notAuthorized();
  ```
- `both`:
  ```ts
  const user = await getCurrentUser(ctx);
  if (!user?._id) return notAuthenticated();
  if (user.status !== 'active' || user.isDeleting) return notAuthorized();
  ```

### Step 5.3 — Convex user functions (if `wants_convex=true` AND NOT `has_user_functions`)

Read `<base_dir>/ref/files/convex-users.md`. Write all 3 files:

- `convex/functions/users/index.ts`
- `convex/functions/users/users.utils.ts`
- `convex/functions/users/internal.ts`

**Adapt if `wants_file_storage=false`:**

- In `internal.ts`: remove the `getConvexImageURL` import and call; return `userData` directly without the `image` computed field
- In `users.utils.ts`: change `image: userData.image || authUser?.pictureUrl || ''` to `image: authUser?.pictureUrl || ''` — the storage URL is gone but the Clerk picture fallback must stay; `IUserWithImage` requires `image: string`

### Step 5.4 — Schema (if `wants_convex=true`)

If `has_convex_schema=false`, create `convex/schema.ts`. If it exists, read it and add missing tables.

Build the `users` table with:

**Always include:**

```ts
clerkId: v.string(),
email: v.string(),
name: v.optional(v.string()),
```

**Include only if `wants_file_storage=true`:**

```ts
imageId: v.optional(v.id('_storage')),
```

**Include based on `user_status_config`:**

- `status` or `both`: `status: v.union(v.literal('active'), v.literal('suspended')),` (adapt values if user specified different ones)
- `soft-delete` or `both`: `isDeleting: v.optional(v.boolean()),`

**Add user's `extra_user_fields`** — map each to the correct Convex validator:

- `string` → `v.string()`
- `optional string` → `v.optional(v.string())`
- `boolean` → `v.boolean()`
- `optional boolean` → `v.optional(v.boolean())`
- `number` → `v.number()`
- union/enum (e.g. `"admin" | "user"`) → `v.union(v.literal("admin"), v.literal("user"))`

**Always include indexes:**

```ts
.index('by_clerkId', ['clerkId'])
.index('by_email', ['email'])
```

For **`additional_tables`**: create each table with sensible fields based on the user's description. Add `_creationTime` is auto-added — never add it manually. Add at least one lookup index per table.

### Step 5.5 — Clerk integration (if `wants_clerk=true`)

Read `<base_dir>/ref/files/clerk-integration.md`. Write the files:

**`middleware.ts`** (if NOT `has_middleware`):

- Replace `/* PUBLIC_ROUTES */` with the user's `extra_public_routes` list as string literals
- Add `afterSignInUrl` and `afterSignUpUrl` if the user specified non-default values:
  ```ts
  export default clerkMiddleware(async (auth, req) => {
    if (!isPublicRoute(req)) {
      await auth.protect();
    }
  });
  ```

**`app/providers.tsx`** (if NOT `has_providers`):

- Write verbatim — no props needed on `<ClerkProvider>`; redirect URLs are set via env vars below.

**`.env.local`** — append these Clerk redirect env vars (create the file if it doesn't exist):

```
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=<after_sign_in_url>
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=<after_sign_up_url>
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=<after_sign_in_url>
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=<after_sign_up_url>
```

Replace `<after_sign_in_url>` and `<after_sign_up_url>` with the values the user gave in Batch B questions 3 and 4 (default: `/dashboard`).

### Step 5.6 — Update `app/layout.tsx`

Read the existing `app/layout.tsx`. Find the `{children}` in the return statement.

If `<Providers>` is not already wrapping `{children}`:

1. Add `import { Providers } from './providers';` to the imports
2. Wrap `{children}` with `<Providers>{children}</Providers>`

Patch minimally — preserve all existing structure (fonts, metadata, html/body tags, classNames).

### Step 5.7 — Hooks (if `wants_convex=true` AND NOT `has_hooks`)

Read `<base_dir>/ref/files/hooks.md`. Write all 4 files verbatim:

- `hooks/convex/use-convex-query.ts`
- `hooks/convex/use-convex-mutation.ts`
- `hooks/convex/use-convex-action.ts`
- `hooks/convex/use-convex-paginated-query.ts`

Create `hooks/convex/` directory if it does not exist.

### Step 5.8 — `.env.local.example`

Create `.env.local.example` at the project root (skip if it already exists). Include all env vars the project needs — omit actual secrets, use placeholder values only:

```
# Convex — auto-written to .env.local by: npx convex dev
NEXT_PUBLIC_CONVEX_URL=https://<your-deployment>.convex.cloud

# Clerk — from https://dashboard.clerk.com → API Keys
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
NEXT_PUBLIC_CLERK_FRONTEND_API_URL=https://<your-clerk-frontend-api-url>

# Clerk redirect URLs
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL=<after_sign_in_url>
NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL=<after_sign_up_url>
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=<after_sign_in_url>
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=<after_sign_up_url>

# Clerk webhook — from Clerk dashboard → Webhooks → signing secret
CLERK_WEBHOOK_SECRET=whsec_...
```

Replace `<after_sign_in_url>` and `<after_sign_up_url>` with the actual values from Batch B (same substitution as Step 5.5; default `/dashboard`).

Append only if `wants_file_storage=true`:
```
# Convex file storage — your deployment's HTTP site URL
CONVEX_SITE_URL=https://<your-deployment>.convex.site
```

### Step 5.9 — HTTP Actions + Clerk webhook (if `wants_convex=true` AND NOT `has_http_router`)

Read `<base_dir>/ref/files/convex-http-actions.md`. Apply sections in order:

**[Always] sections** — write all 3 files verbatim:

- `convex/httpActions/utils.httpActions.ts`
- `convex/httpActions/index.ts`
- `convex/http.ts`

**[If wants_file_storage=true] sections** — apply after the always sections:

1. Write `convex/httpActions/convexStorage/image.ts` verbatim.
2. Patch `convex/httpActions/index.ts`: add the `convexStorageHttpActions` import and merge `convexStorage` into `httpActionsHandlers`.
3. Patch `convex/http.ts`: add the `GET /convex/storage/get-image` route before `export default http;`.

**[If wants_clerk=true] sections** — apply immediately after the file storage sections (or always sections if `wants_file_storage=false`):

1. Write `convex/httpActions/clerk/clerk.httpActions.ts` verbatim (skip if `has_webhook_handler=true`).

2. Append `upsertUserFromClerk` and `deleteUserByClerkId` to `convex/functions/users/internal.ts` — read the file first; skip any mutation that is already present:
   - Add `internalMutation` to the existing import from `../../_generated/server`
   - Append both mutations below `userByClerkIdInternal`
   - **Adapt `upsertUserFromClerk` insert payload** based on `user_status_config`:
     - `status` or `both`: add `status: 'active' as const`
     - `soft-delete` or `both`: nothing — `isDeleting` is absent on new users by design
   - **Adapt for required `extra_user_fields`**: non-optional fields need a sensible insert default — `''` for string, `false` for boolean, `0` for number, first literal for union/enum. Add `// ⚠️ default for required field` comment next to each.
   - **Do not adapt `deleteUserByClerkId`** — write it verbatim; deletion logic is intentionally left for the developer to implement.

3. Patch `convex/httpActions/index.ts` per the ref file instructions (add import + populate `httpActionsHandlers`).

4. Patch `convex/http.ts` per the ref file instructions (add the `/api/webhooks/clerk` route before `export default http`).

If `convex/http.ts` already exists, skip this entire step — do not overwrite existing HTTP routing.

---

## Phase 6 — Write Rules and CLAUDE.md

### Step 6.1 — Rules files

Create `.claude/rules/` directory if it does not exist.

For each rule file, read the ref file and write it verbatim to `.claude/rules/`. If the file already exists, read it first — append only if the content is not already there.

| Read from                        | Write to                                                |
| -------------------------------- | ------------------------------------------------------- |
| `<base_dir>/ref/rules/convex.md` | `.claude/rules/convex.md` (only if `wants_convex=true`) |
| `<base_dir>/ref/rules/hooks.md`  | `.claude/rules/hooks.md` (only if `wants_convex=true`)  |
| `<base_dir>/ref/rules/nextjs.md` | `.claude/rules/nextjs.md`                               |
| `<base_dir>/ref/rules/clerk.md`  | `.claude/rules/clerk.md` (only if `wants_clerk=true`)   |

### Step 6.2 — CLAUDE.md

Read `<base_dir>/ref/claude-md-section.md`.

- If `CLAUDE.md` does not exist: create it and write the section content.
- If it exists: read it, then **append** the section. Never overwrite existing content.
  Check that the `## Stack` heading is not already present — if it is, skip to avoid duplication.

---

## Phase 7 — Report

Output a clean, scannable summary:

```
✅ Setup complete  [Scenario A / B / C]

INSTALLED:
  • @clerk/nextjs vX.X.X
  • convex vX.X.X
  • http-status vX.X.X
  [or "Nothing new — all packages already present"]

CREATED:
  convex/utils/httpStatusCode.ts
  convex/types/common.ts
  convex/constants/pagination.ts
  convex/validations/common.ts
  convex/types/convex-types.ts
  convex/utils/common.ts
  convex/utils/middlewares/convexMiddleware.ts    [middleware variant: A / B / C]
  convex/utils/middlewares/convexPublicMiddleware.ts
  convex/functions/users/users.utils.ts
  convex/functions/users/internal.ts
  convex/schema.ts                               [tables: users, ...]
  middleware.ts                                  [public routes: /, /sign-in, /sign-up, ...]
  app/providers.tsx
  hooks/convex/use-convex-query.ts
  hooks/convex/use-convex-mutation.ts
  hooks/convex/use-convex-action.ts
  hooks/convex/use-convex-paginated-query.ts
  convex/httpActions/utils.httpActions.ts
  convex/httpActions/index.ts                        [handlers registered based on config]
  convex/httpActions/convexStorage/image.ts          [only if wants_file_storage]
  convex/httpActions/clerk/clerk.httpActions.ts      [only if wants_clerk]
  convex/http.ts                                     [GET /convex/storage/get-image if wants_file_storage | POST /api/webhooks/clerk if wants_clerk]
  .claude/rules/convex.md
  .claude/rules/hooks.md
  .claude/rules/nextjs.md
  .claude/rules/clerk.md
  [Mark SKIPPED — already existed for any file not created]

UPDATED:
  app/layout.tsx     (wrapped {children} with <Providers>)
  CLAUDE.md          (appended stack conventions)

⚠️  REQUIRED NEXT STEPS — the app will not work until these are done:

1. Initialize Convex:
     npx convex dev
   → This writes NEXT_PUBLIC_CONVEX_URL to .env.local automatically.

2. Create a Clerk application at https://dashboard.clerk.com
   Copy your keys and add to .env.local:
     NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
     CLERK_SECRET_KEY=sk_...
     NEXT_PUBLIC_CLERK_FRONTEND_API_URL=https://<your-clerk-frontend-api-url>

3. In Convex dashboard → Settings → Environment Variables, add:
     NEXT_PUBLIC_CLERK_FRONTEND_API_URL=https://<your-clerk-frontend-api-url>

4. Register the Clerk webhook (the handler at /api/webhooks/clerk is already created):
   a) In Clerk dashboard → Webhooks, add an endpoint:
        URL: https://<your-deployment>.convex.site/api/webhooks/clerk
        Events: user.created, user.updated, user.deleted
   b) Copy the signing secret → add to .env.local:
        CLERK_WEBHOOK_SECRET=whsec_...
   c) Add to Convex dashboard → Settings → Environment Variables:
        CLERK_WEBHOOK_SECRET=whsec_...
   [Only if using file storage:]
        CONVEX_SITE_URL=https://<your-deployment>.convex.site

5. Create Clerk auth pages (if not already present):
   ```
   app/sign-in/[[...sign-in]]/page.tsx   → render <SignIn /> from '@clerk/nextjs'
   app/sign-up/[[...sign-up]]/page.tsx   → render <SignUp /> from '@clerk/nextjs'
   ```

6. Run dev servers (two terminals):
     npx convex dev
     npm run dev
```

If anything was skipped or failed, list it clearly at the bottom with the reason.
