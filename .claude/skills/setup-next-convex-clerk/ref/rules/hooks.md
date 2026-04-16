---
description: Custom hook conventions — never use Convex hooks directly, always use wrappers, domain hook patterns
globs: hooks/**/*.ts,hooks/**/*.tsx,app/**/_hooks/**/*.ts,app/**/_hooks/**/*.tsx
---

# Hook Rules

## Never Use Convex Hooks Directly in Components

**Never** call `useQuery`, `useMutation`, or `useAction` directly in components.
Always use the custom wrapper hooks from `hooks/convex/`:

| Use this | Instead of | Returns |
|---|---|---|
| `useConvexQuery(fn, args)` | `useQuery` | `{ data, error, isLoading }` |
| `useConvexMutation(fn, opts?)` | `useMutation` | `{ data, error, isLoading, isSuccess, isError, isSettled, mutate, reset }` |
| `useConvexAction(fn, opts?)` | `useAction` | same shape but `call` instead of `mutate` |
| `useConvexPaginatedQuery(fn, args)` | `usePaginatedQuery` | `{ data, meta, error, isLoading, pagination }` |

```ts
// ✅ Correct
import useConvexQuery from '@/hooks/convex/use-convex-query';
const { data, isLoading } = useConvexQuery(api.functions.items.index.readItems, { sortOrder: 'desc' });

// ❌ Wrong
import { useQuery } from 'convex/react';
const result = useQuery(api.functions.items.index.readItems, { sortOrder: 'desc' });
```

## Response Unwrapping

`useConvexQuery` automatically unwraps `IConvexResponse` — `data` is already the payload, never the envelope. Do not check `.success` on hook return values.

`useConvexMutation` / `useConvexAction` `onSuccess` fires **only** when `result.success === true`. The callback receives the full `IConvexResult` envelope — access payload via `data.data`.

## Domain Hooks

Wrap core hooks in domain-specific hooks to expose typed, reusable APIs to components:

```ts
// hooks/use-create-item.ts
'use client';
import { api } from '@/convex/_generated/api';
import { useConvexMutation } from './convex/use-convex-mutation';

export function useCreateItem() {
  return useConvexMutation(api.functions.items.index.createItem, {
    onSuccess: (data) => {
      // data.data is the created item — show toast, redirect, etc.
    },
  });
}
```

## Paginated Queries

Backend queries for `useConvexPaginatedQuery` must return `IConvexResponse` with `meta: { cursor, isLastPage, limit, sortOrder }`. Use `.paginate()` on the backend — never return raw `PaginationResult<T>`.

```ts
// component
const { data, isLoading, pagination } = useConvexPaginatedQuery(
  api.functions.items.index.readItems,
  { sortOrder: 'desc' },
);
// pagination.handleNextPage(), handlePrevPage(), handleFirstPage()
// pagination.handleLimitChange(20), handleSortOrderChange('asc')
// pagination.hasPrevPage, pagination.isLastPage
```

## File Location Conventions

| Location | Contains |
|---|---|
| `hooks/convex/` | Core wrapper hooks — never edit these directly |
| `hooks/` root | Reusable domain hooks (`use-create-item.ts`, `use-current-user.ts`) |
| `app/<route>/_hooks/` | Page-local hooks — never import from another route |

- One hook per file
- `use-kebab-case.ts` naming
- Always type return values explicitly — never return `any`
