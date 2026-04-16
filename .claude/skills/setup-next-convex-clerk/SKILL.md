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
<base_dir>/ref/files/convex-utils.md        → 6 utility files (httpStatusCode, types, constants, validations, convex-types, common)
<base_dir>/ref/files/convex-middlewares.md  → convexMiddleware + convexPublicMiddleware
<base_dir>/ref/files/convex-users.md        → users.utils.ts + users/internal.ts
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

Ask **one** AskUserQuestion call with all applicable items. Skip items that are already installed.

```
I've scanned your project. Here's what I found:
[Brief bullet list — e.g. "✅ Next.js 15 | ❌ Clerk not installed | ❌ Convex not installed | ✅ Tailwind CSS"]

Please answer these to continue:

[Include only lines that apply:]
1. 🔐 Clerk — Will you use Clerk for authentication? (yes / no)
2. 🗄️  Convex — Will you use Convex as your real-time backend/database? (yes / no)
3. 🎨 ShadCN/UI — Will you use ShadCN/UI for components? (yes / no)
4. 📝 App — Briefly describe what this app does. (One line — helps design schema and rules)
```

Record: `wants_clerk`, `wants_convex`, `wants_shadcn`, `app_description`

If already installed, set `wants_clerk=true` / `wants_convex=true` automatically without asking.

**If Scenario C:** Skip Batch A entirely. Go to Batch B and Batch C.

---

## Phase 2 — Batch B: Clerk Questions

**Only ask if `wants_clerk=true`.** One AskUserQuestion call.

```
A few Clerk-specific questions:

1. 🔓 Public routes — Which routes should be accessible WITHOUT login?
   Already included by default: /, /sign-in, /sign-up, /api/webhooks
   List any extra paths (e.g. /about, /pricing, /blog) — or type "none".

2. 🏢 Clerk Organizations — Does your app need multi-tenant organization support? (yes / no)
   (Most apps don't — answer "no" if unsure)

3. ↩️  After sign-in — Where should users land after signing in?
   Default: /dashboard — or specify a different path.

4. ↩️  After sign-up — Where should users land after signing up?
   Default: /dashboard — or specify a different path (often /onboarding for new users).
```

Record: `extra_public_routes` (list), `wants_orgs` (bool), `after_sign_in_url`, `after_sign_up_url`

---

## Phase 3 — Batch C: Convex Questions

**Only ask if `wants_convex=true`.** One AskUserQuestion call.

```
A few Convex-specific questions:

1. 👤 Extra user fields — Beyond the defaults (email, name, imageId), what extra fields do your users need?
   Examples:
     role: "admin" | "user"
     plan: "free" | "pro" | "enterprise"
     onboardingComplete: boolean
     bio: string (optional)
   → List them with types, or type "none" for defaults only.

2. 🔄 User status / soft-delete — Does your users table need:
   a) A status field (e.g. "active" | "suspended") that gates access?
   b) A soft-delete field (e.g. isDeleting: boolean)?
   → Answer "a", "b", "both", or "none". This affects the auth middleware logic.

3. 🗃️  Additional tables — What other data tables do you know you'll need?
   Example: posts (title, content, authorId, status: "published" | "draft")
   → Describe each, or type "none".

4. 🖼️  File storage — Does this app need Convex file storage for uploads (avatars, images, etc.)?
   (yes / no — if "no", imageId is removed from the users table)
```

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

**Adapt `convexMiddleware.ts` Step 2** based on `user_status_config`:

- `none` → use Variant A: `if (!user?._id) { return notAuthorized(); }`
- `status` → use Variant B: `if (!user?._id || user?.status !== 'active') { return notAuthorized(); }`
- `soft-delete` → use Variant A + add: `if (!user?._id || user?.isDeleting) { return notAuthorized(); }`
- `both` → use Variant C: `if (!user?._id || user?.status !== 'active' || user?.isDeleting) { return notAuthorized(); }`

### Step 5.3 — Convex user functions (if `wants_convex=true` AND NOT `has_user_functions`)

Read `<base_dir>/ref/files/convex-users.md`. Write both files:

- `convex/functions/users/users.utils.ts`
- `convex/functions/users/internal.ts`

**Adapt if `wants_file_storage=false`:**

- In `internal.ts`: remove the `getConvexImageURL` import and call; return `userData` directly without the `image` computed field
- In `users.utils.ts`: remove the `image` field from the spread return

### Step 5.4 — Schema (if `wants_convex=true`)

If `has_convex_schema=false`, create `convex/schema.ts`. If it exists, read it and add missing tables.

Build the `users` table with:

**Always include:**

```ts
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

**Always include index:**

```ts
.index('by_email', ['email'])
```

For **`additional_tables`**: create each table with sensible fields based on the user's description. Add `_creationTime` is auto-added — never add it manually. Add at least one lookup index per table.

### Step 5.5 — Clerk integration (if `wants_clerk=true`)

Read `<base_dir>/ref/files/clerk-integration.md`. Write the files:

**`middleware.ts`** (if NOT `has_middleware`):

- Replace `/* PUBLIC_ROUTES */` with the user's `extra_public_routes` list as string literals
- Add `afterSignInUrl` and `afterSignOutUrl` if the user specified non-default values:
  ```ts
  export default clerkMiddleware(async (auth, req) => {
    if (!isPublicRoute(req)) {
      await auth.protect();
    }
  });
  ```

**`app/providers.tsx`** (if NOT `has_providers`):

- If `wants_orgs=true`, add `afterSignInUrl` and `afterSignUpUrl` props to `<ClerkProvider>`:
  ```tsx
  <ClerkProvider afterSignInUrl="<after_sign_in_url>" afterSignUpUrl="<after_sign_up_url>">
  ```

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
     CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>

3. In Convex dashboard → Settings → Environment Variables, add:
     CLERK_JWT_ISSUER_DOMAIN=https://<your-clerk-frontend-api-url>

4. In Clerk dashboard → Webhooks, add an endpoint:
     URL: <your-convex-site-url>/api/webhooks/clerk
     Events: user.created, user.updated
   Copy the signing secret and add to .env.local:
     CLERK_WEBHOOK_SECRET=whsec_...
   [Only if using file storage — add to Convex env vars:]
     CONVEX_SITE_URL=https://<your-deployment>.convex.site

5. Run dev servers (two terminals):
     npx convex dev
     npm run dev
```

If anything was skipped or failed, list it clearly at the bottom with the reason.
