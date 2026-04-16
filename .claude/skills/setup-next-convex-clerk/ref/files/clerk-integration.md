# Clerk Integration Files

Write each file below to the exact path shown.

> **Adaptation for `middleware.ts`:**
> Replace `/* PUBLIC_ROUTES */` with the user's confirmed public routes list.
> Always include: `'/'`, `'/sign-in(.*)'`, `'/sign-up(.*)'`, `'/api/webhooks(.*)'`.
> Append any extra routes the user specified.

---

## `middleware.ts` — project root

```ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher([
  '/',
  '/sign-in(.*)',
  '/sign-up(.*)',
  '/api/webhooks(.*)',
  /* PUBLIC_ROUTES — insert user-specified extra routes here, e.g.: '/about', '/pricing' */
]);

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: [
    '/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)',
    '/(api|trpc)(.*)',
  ],
};
```

---

## `app/providers.tsx`

```tsx
'use client';

import { ClerkProvider, useAuth } from '@clerk/nextjs';
import { ConvexProviderWithClerk } from 'convex/react-clerk';
import { ConvexReactClient } from 'convex/react';

// Created at module scope — not inside the component — to avoid re-creating on every render
const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <ConvexProviderWithClerk client={convex} useAuth={useAuth}>
        {children}
      </ConvexProviderWithClerk>
    </ClerkProvider>
  );
}
```
