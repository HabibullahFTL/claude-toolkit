# Custom Convex Hooks

Write each file below to the exact path shown. Create the `hooks/convex/` directory if it does not exist.
These hooks wrap Convex's native hooks — components must never call the native hooks directly.

---

## `hooks/convex/use-convex-query.ts`

```ts
'use client';

import {
  IConvexErrorResponse,
  IConvexResponse,
} from '@/convex/types/convex-types';
import { OptionalRestArgsOrSkip, useQuery } from 'convex/react';
import { DefaultFunctionArgs, FunctionReference } from 'convex/server';
import { useMemo } from 'react';

type IQuery<TData, TArgs extends DefaultFunctionArgs> = FunctionReference<
  'query',
  'public',
  TArgs,
  IConvexErrorResponse | IConvexResponse<TData>
>;

const useConvexQuery = <
  TData,
  TArgs extends DefaultFunctionArgs,
  TArgument extends OptionalRestArgsOrSkip<IQuery<TData, TArgs>>,
>(
  query: IQuery<TData, TArgs>,
  ...args: TArgument
) => {
  const result = useQuery(query, ...(args || []));
  const data = useMemo(
    () => (result?.success === true ? result.data : undefined),
    [result],
  );
  const error = useMemo(
    () => (result?.success === false ? result : undefined),
    [result],
  );
  return { data, error, isLoading: result === undefined };
};

export default useConvexQuery;
```

---

## `hooks/convex/use-convex-mutation.ts`

```ts
'use client';

import { useMutation } from 'convex/react';
import { FunctionReference } from 'convex/server';
import { useCallback, useEffect, useRef, useState } from 'react';

interface IHookOptions<TData> {
  // Fires ONLY when result.success === true.
  // `data` is the full IConvexResult envelope — access payload via data.data
  onSuccess?: (data: TData) => void;
  onError?: (error: Error) => void;
  onSettled?: () => void;
}

const makeInitialState = <T>() => ({
  isLoading: false,
  isError: false,
  isSuccess: false,
  isSettled: false,
  error: null as Error | null,
  data: null as T | null,
});

export function useConvexMutation<
  TFn extends FunctionReference<'mutation'>,
  TArgs extends Parameters<ReturnType<typeof useMutation<TFn>>>,
  TReturn extends Awaited<ReturnType<ReturnType<typeof useMutation<TFn>>>>,
>(fn: TFn, options?: IHookOptions<TReturn>) {
  const mutation = useMutation(fn);
  const [state, setState] = useState(() => makeInitialState<TReturn>());

  // Keep callback refs in sync so stale closures never fire old handlers
  const onSuccessRef = useRef(options?.onSuccess);
  const onErrorRef = useRef(options?.onError);
  const onSettledRef = useRef(options?.onSettled);
  useEffect(() => { onSuccessRef.current = options?.onSuccess; }, [options?.onSuccess]);
  useEffect(() => { onErrorRef.current = options?.onError; }, [options?.onError]);
  useEffect(() => { onSettledRef.current = options?.onSettled; }, [options?.onSettled]);

  const reset = useCallback(() => {
    setState(makeInitialState<TReturn>());
  }, []);

  const mutate = useCallback(
    async (...args: TArgs): Promise<TReturn> => {
      setState({ ...makeInitialState<TReturn>(), isLoading: true });
      try {
        const result = await mutation(...args);
        // A Convex-level error (success: false) is NOT a thrown error — handle both paths
        setState((prev) => ({
          ...prev,
          isSuccess: result?.success === true,
          isError: result?.success === false,
          data: result,
        }));
        if (result?.success === true) onSuccessRef.current?.(result);
        return result;
      } catch (err) {
        const error = err as Error;
        setState((prev) => ({ ...prev, isError: true, error }));
        onErrorRef.current?.(error);
        throw error;
      } finally {
        setState((prev) => ({ ...prev, isLoading: false, isSettled: true }));
        onSettledRef.current?.();
      }
    },
    [mutation],
  );

  return { ...state, mutate, reset };
}
```

---

## `hooks/convex/use-convex-action.ts`

```ts
'use client';

import { useAction } from 'convex/react';
import { FunctionReference } from 'convex/server';
import { useCallback, useEffect, useRef, useState } from 'react';

interface IHookOptions<TData> {
  // Fires ONLY when result.success === true.
  // `data` is the full IConvexResult envelope — access payload via data.data
  onSuccess?: (data: TData) => void;
  onError?: (error: Error) => void;
  onSettled?: () => void;
}

const makeInitialState = <T>() => ({
  isLoading: false,
  isError: false,
  isSuccess: false,
  isSettled: false,
  error: null as Error | null,
  data: null as T | null,
});

export function useConvexAction<
  TFn extends FunctionReference<'action'>,
  TArgs extends Parameters<ReturnType<typeof useAction<TFn>>>,
  TReturn extends Awaited<ReturnType<ReturnType<typeof useAction<TFn>>>>,
>(fn: TFn, options?: IHookOptions<TReturn>) {
  const action = useAction(fn);
  const [state, setState] = useState(() => makeInitialState<TReturn>());

  const onSuccessRef = useRef(options?.onSuccess);
  const onErrorRef = useRef(options?.onError);
  const onSettledRef = useRef(options?.onSettled);
  useEffect(() => { onSuccessRef.current = options?.onSuccess; }, [options?.onSuccess]);
  useEffect(() => { onErrorRef.current = options?.onError; }, [options?.onError]);
  useEffect(() => { onSettledRef.current = options?.onSettled; }, [options?.onSettled]);

  const reset = useCallback(() => {
    setState(makeInitialState<TReturn>());
  }, []);

  const call = useCallback(
    async (...args: TArgs): Promise<TReturn> => {
      setState({ ...makeInitialState<TReturn>(), isLoading: true });
      try {
        const result = await action(...args);
        setState((prev) => ({
          ...prev,
          isSuccess: result?.success === true,
          isError: result?.success === false,
          data: result,
        }));
        if (result?.success === true) onSuccessRef.current?.(result);
        return result;
      } catch (err) {
        const error = err as Error;
        setState((prev) => ({ ...prev, isError: true, error }));
        onErrorRef.current?.(error);
        throw error;
      } finally {
        setState((prev) => ({ ...prev, isLoading: false, isSettled: true }));
        onSettledRef.current?.();
      }
    },
    [action],
  );

  return { ...state, call, reset };
}
```

---

## `hooks/convex/use-convex-paginated-query.ts`

```ts
'use client';

import {
  DEFAULT_PAGE_LIMIT,
  DEFAULT_SORT_ORDER,
  defaultSortOrders,
} from '@/convex/constants/pagination';
import { ISortOrder } from '@/convex/types/common';
import {
  IConvexErrorResponse,
  IConvexResponse,
} from '@/convex/types/convex-types';
import { OptionalRestArgsOrSkip, useQuery } from 'convex/react';
import { DefaultFunctionArgs, FunctionReference } from 'convex/server';
import { useMemo, useState } from 'react';

type IQuery<TData, TArgs extends DefaultFunctionArgs> = FunctionReference<
  'query',
  'public',
  TArgs,
  IConvexErrorResponse | IConvexResponse<TData>,
  string | undefined
>;

type PaginationArgs = {
  cursor?: string;
  limit?: number;
  sortOrder?: ISortOrder;
  search?: string;
};

export const useConvexPaginatedQuery = <
  TData,
  TArgs extends DefaultFunctionArgs,
  TArgument extends OptionalRestArgsOrSkip<IQuery<TData, TArgs>>,
>(
  query: IQuery<TData, TArgs>,
  args: Partial<TArgs>,
) => {
  const {
    limit: initialLimit = DEFAULT_PAGE_LIMIT,
    search = '',
    sortOrder: initialSortOrder = DEFAULT_SORT_ORDER,
  } = (args ?? {}) as PaginationArgs;

  const [limit, setLimit] = useState(initialLimit);
  const [sortOrder, setSortOrder] = useState(initialSortOrder);
  const [cursorStack, setCursorStack] = useState<(string | undefined)[]>([]);
  const [currentCursor, setCurrentCursor] = useState<string>();

  const result = useQuery(
    query,
    ...([
      {
        ...args,
        limit,
        cursor: currentCursor,
        sortOrder,
        ...(search ? { search } : {}),
      },
    ] as unknown as TArgument),
  );

  const successResult = useMemo(
    () => (result && result.success === true ? result : undefined),
    [result],
  );

  const data = useMemo(() => successResult?.data ?? [], [successResult]) as TData;
  const meta = useMemo(() => successResult?.meta, [successResult]);
  const error = useMemo(
    () => (result && result.success === false ? result : undefined),
    [result],
  );

  const handleNextPage = () => {
    if (meta?.cursor) {
      setCursorStack((prev) => [...prev, currentCursor!]);
      setCurrentCursor(meta?.cursor);
    }
  };

  const handlePrevPage = () => {
    const newStack = [...cursorStack];
    const prevCursor = newStack.pop(); // undefined = first page — do NOT coerce to ''
    setCursorStack(newStack);
    setCurrentCursor(prevCursor);
  };

  const handleFirstPage = () => {
    setCursorStack([]);
    setCurrentCursor('');
  };

  const handleLimitChange = (value: number) => {
    if (limit === value) return;
    setLimit(value);
    setCursorStack([]);
    setCurrentCursor('');
  };

  const handleSortOrderChange = (value: string) => {
    if (value === sortOrder || !defaultSortOrders.includes(value as ISortOrder)) return;
    setSortOrder(value as ISortOrder);
    setCursorStack([]);
    setCurrentCursor('');
  };

  return {
    data,
    meta,
    error,
    isLoading: result === undefined,
    pagination: {
      limit,
      sortOrder,
      hasPrevPage: cursorStack.length > 0,
      isLastPage: meta?.isLastPage ?? false,
      handleNextPage,
      handlePrevPage,
      handleFirstPage,
      handleLimitChange,
      handleSortOrderChange,
    },
  };
};
```
