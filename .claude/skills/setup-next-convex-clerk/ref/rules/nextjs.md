---
description: Next.js App Router conventions — Server Components default, page composition, preloadQuery pattern, metadata, component structure
globs: app/**/*.ts,app/**/*.tsx,components/**/*.ts,components/**/*.tsx
---

# Next.js App Router Rules

## Server Components by Default

Add `"use client"` **only** when the component uses:
- React hooks (`useState`, `useEffect`, `useRef`, etc.)
- Browser APIs (`window`, `localStorage`, etc.)
- Convex real-time hooks (`useConvexQuery`, `useConvexMutation`, etc.)
- Event handlers that cannot be Server Actions

**Never** add `"use client"` to layouts or route segments unless absolutely necessary.

## page.tsx — Compose Only

`page.tsx` must be a Server Component. It **composes** — no business logic, no data fetching beyond `preloadQuery`, no inline JSX beyond composition.

```tsx
// ✅ Correct — compose only
import { api } from '@/convex/_generated/api';
import { preloadQuery } from 'convex/nextjs';
import DashboardView from './_components/dashboard-view';

const DashboardPage = async () => {
  const preloadedItems = await preloadQuery(
    api.functions.items.index.readItems,
    { sortOrder: 'desc' },
  );
  return <DashboardView preloadedItems={preloadedItems} />;
};
export default DashboardPage;

// ❌ Wrong — logic and JSX in page
const DashboardPage = async () => {
  const items = await fetchSomething();
  return <div>{items.map(i => <div key={i._id}>{i.name}</div>)}</div>;
};
```

## SSR Data — preloadQuery / usePreloadedQuery

Pass server-loaded data to Client Components to avoid waterfalls:

```tsx
// Server: page.tsx
const preloaded = await preloadQuery(api.functions.items.index.readItems, args);
return <View preloadedItems={preloaded} />;

// Client: _components/view.tsx
'use client';
import { usePreloadedQuery, Preloaded } from 'convex/react';
import { api } from '@/convex/_generated/api';

interface Props {
  preloadedItems: Preloaded<typeof api.functions.items.index.readItems>;
}
export default function View({ preloadedItems }: Props) {
  const result = usePreloadedQuery(preloadedItems);
  // result is IConvexResponse | IConvexErrorResponse — always check .success
  const items = result?.success ? result.data : [];
}
```

**Always unwrap via `result?.success`** — never access `.data` without checking `.success` first.

## Required Per Route Segment

Every meaningful route should have:
- `loading.tsx` — shown while the page suspends (Suspense boundary)
- `error.tsx` — unhandled errors (`'use client'` required)
- `generateMetadata` or exported `metadata` — for all public-facing pages

## Component Folder Structure

```
app/<route>/
  page.tsx                 ← Server Component, compose only
  loading.tsx              ← Suspense fallback
  error.tsx                ← Error boundary (must be 'use client')
  layout.tsx               ← Providers + shell only; no data fetching
  _components/             ← page-local (never import from another route)
    <feature>/
      index.tsx            ← public face of the component
      <sub-part>.tsx
  _utils/                  ← page-local helpers
  _hooks/                  ← page-local hooks
```

`_components/`, `_utils/`, `_hooks/` are **private to their route**. Never import them from another route. If something is needed in multiple routes, move it to `components/shared/`.

## Reusable Components

```
components/
  ui/         ← ShadCN generated — never edit directly; wrap to extend
  shared/     ← cross-route reusable components
```

**Never edit ShadCN `ui/` components directly.** Create a wrapper component to extend them.
**Always use radix-ui primitives. Never use base-ui.**

## Utilities and Assets

- `next/image` for all images — never raw `<img>`
- `next/link` for all internal navigation — never `<a href>`
- `next/font` for fonts — never CSS `@import` for fonts
- `cn()` from `lib/utils.ts` for all conditional class composition (clsx + tailwind-merge)

## Naming Conventions

| Target | Convention |
|---|---|
| Folders and files | `kebab-case` |
| React components | `PascalCase` (named export preferred) |
| Hooks | `useCamelCase` |
| Constants | `SCREAMING_SNAKE_CASE` |
| Types / interfaces | `PascalCase`, prefix interfaces with `I` |
