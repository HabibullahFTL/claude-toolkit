# Convex Utility Files

Write each file below to the exact path shown. Create parent directories as needed.
Skip any file that already exists and has content — read it and patch only what is missing.
These files must be created in the order shown (each depends on the one above it).

---

## `convex/utils/httpStatusCode.ts`

```ts
import httpStatus from 'http-status';

const HttpStatusCodes = httpStatus;

export default HttpStatusCodes;
```

---

## `convex/types/common.ts`

```ts
export type ISortOrder = 'asc' | 'desc';
```

---

## `convex/constants/pagination.ts`

```ts
export const DEFAULT_PAGE_LIMIT = 10;
export const DEFAULT_SORT_ORDER = 'desc' as const;
export const defaultSortOrders = ['asc', 'desc'] as const;
```

---

## `convex/validations/common.ts`

```ts
import { z } from 'zod';
import { Id } from '../_generated/dataModel';
import { DEFAULT_PAGE_LIMIT } from '../constants/pagination';

export const emailZodSchema = z
  .string({ required_error: 'Email address is required.' })
  .min(1, 'Email address is required')
  .email('Invalid email address.');

export const storageIdZodSchema = z
  .string({ required_error: 'Storage ID is required.' })
  .min(1, 'Storage ID is required.') as unknown as z.ZodType<Id<'_storage'>>;

export const userIdZodSchema = z
  .string({ required_error: 'User ID is required.' })
  .min(1, 'User ID is required.') as unknown as z.ZodType<Id<'users'>>;

export const sortOrderZodSchema = z
  .union([z.literal('asc'), z.literal('desc')])
  .default('desc');

// Use this as the zodSchema for any paginated query
export const convexPaginationQueryZodSchema = z.object({
  search: z.string().optional(),
  sortOrder: sortOrderZodSchema,
  cursor: z.string().optional(),
  limit: z.coerce
    .number()
    .optional()
    .default(DEFAULT_PAGE_LIMIT)
    .refine((val) => val > 0, { message: 'Limit must be a positive number' }),
});

// Schema for the meta object returned in paginated responses.
// Intentionally NOT extended from convexPaginationQueryZodSchema —
// `search` is a request field, not a response field.
export const convexResponseMetaZodSchema = z.object({
  cursor: z.string().optional(),     // continueCursor from .paginate(); absent on last page
  isLastPage: z.boolean(),
  limit: z.coerce.number().optional().default(Number(DEFAULT_PAGE_LIMIT)),
  sortOrder: sortOrderZodSchema,
});
```

---

## `convex/types/convex-types.ts`

```ts
import { z } from 'zod';
import { MutationCtx, QueryCtx, ActionCtx } from '../_generated/server';
import { Doc } from '../_generated/dataModel';
import { convexResponseMetaZodSchema } from '../validations/common';

// Union of all Convex context types — use for middleware generics
export type IConvexCtx = QueryCtx | MutationCtx | ActionCtx;
export type IConvexResponseMeta = z.infer<typeof convexResponseMetaZodSchema>;

// Shape of a success response from generateConvexSuccessResponse
export interface IConvexResponse<TData = unknown, TMeta = unknown> {
  success: true;
  statusCode: number;
  message: string;
  data: TData;
  meta?: TMeta;
}

// Shape of an error response from generateConvexErrorResponse
export interface IConvexErrorResponse {
  success: false;
  statusCode: number;
  message: string;
  errorSources?: { path: string; message: string }[];
}

// Union — what every Convex function handler returns
export type IConvexResult<TData = unknown, TMeta = unknown> =
  | IConvexResponse<TData, TMeta>
  | IConvexErrorResponse;

// Extend with actual users table fields once schema.ts is defined
export type IUser = Doc<'users'>;
```

---

## `convex/utils/common.ts`

```ts
import { ZodError } from 'zod';
import { Id } from '../_generated/dataModel';
import { IConvexResponse, IConvexErrorResponse } from '../types/convex-types';
import HttpStatusCodes from './httpStatusCode';

// Returns a typed success envelope
export function generateConvexSuccessResponse<TData, TMeta = undefined>(
  statusCode: number,
  message: string,
  data: TData,
  meta?: TMeta,
): IConvexResponse<TData, TMeta> {
  return {
    success: true,
    statusCode,
    message,
    data,
    ...(meta !== undefined ? { meta } : {}),
  };
}

// Returns a typed error envelope — never throw; always return this
export function generateConvexErrorResponse(
  statusCode: number,
  message: string,
  errorSources?: { path: string; message: string }[],
): IConvexErrorResponse {
  return {
    success: false,
    statusCode,
    message,
    ...(errorSources ? { errorSources } : {}),
  };
}

// Converts a ZodError into a structured error envelope
export function handleConvexZodError(error: ZodError): IConvexErrorResponse {
  const issues = error.issues ?? [];
  const errorSources = issues.map((issue) => ({
    path: String(issue.path[issue.path.length - 1] ?? ''),
    message: issue.message,
  }));
  const message =
    issues.length === 1
      ? issues[0].message
      : `Invalid input in ${errorSources.map((s) => `'${s.path}'`).join(', ')} fields.`;
  return {
    success: false,
    statusCode: HttpStatusCodes.BAD_REQUEST,
    message,
    errorSources,
  };
}

// Constructs a public URL for a file stored in Convex storage.
// ⚠️ CONVEX_SITE_URL must be set manually in the Convex dashboard (Settings → Environment Variables).
// Format: https://<your-deployment-slug>.convex.site
// Remove this function if the project does not use Convex file storage.
export function getConvexImageURL(storageId?: Id<'_storage'>): string {
  if (!storageId || !process.env.CONVEX_SITE_URL) {
    return '';
  }
  const url = new URL(`${process.env.CONVEX_SITE_URL}/convex/storage/get-image`);
  url.searchParams.set('storageId', storageId);
  return url.href;
}
```
