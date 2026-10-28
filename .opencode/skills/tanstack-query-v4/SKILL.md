---
name: tanstack-query-v4
description: Data fetching with @tanstack/react-query 4.24. Use when writing useQuery, useMutation, query keys, caching, invalidation, infinite queries, or devtools. This repo is v4 - v5 APIs (gcTime, isPending, placeholderData keepPreviousData) do NOT apply.
---

# TanStack Query v4 (pinned: @tanstack/react-query@4.24, devtools@4.28)

Offline-first. v4-only rules — do not use v5 syntax.

## 1. Version traps (v4 vs v5)

| Concept                    | v4 (THIS REPO)                        | v5 (DO NOT USE)                     |
| -------------------------- | ------------------------------------- | ----------------------------------- |
| GC time                    | `cacheTime`                           | `gcTime`                            |
| Loading flag               | `isLoading`                           | `isPending`                         |
| Keep screen on page change | `keepPreviousData: true`              | `placeholderData: keepPreviousData` |
| Query errors in useQuery   | `onSuccess/onError/onSettled` allowed | removed from useQuery               |
| Axios errors               | `isAxiosError` narrowing              | same, but shape differs             |

```tsx
// CORRECT v4 pagination
useQuery({
  queryKey: ['docs', page],
  queryFn: () => fetchDocs(page),
  keepPreviousData: true, // v4 way
  staleTime: 30_000,
  cacheTime: 5 * 60_000,
})
```

## 2. Query keys: factory + hierarchy

```tsx
export const docKeys = {
  all: ['docs'] as const,
  list: (filters: DocFilters) => [...docKeys.all, 'list', filters] as const,
  detail: (id: string) => [...docKeys.all, 'detail', id] as const,
}

useQuery({
  queryKey: docKeys.detail(id),
  queryFn: () => fetchDoc(id),
  enabled: !!id,
})
```

Keys always arrays, generic → specific, colocated with the feature. `enabled: !!id` for dependent queries.

## 3. Client setup (global defaults)

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const client = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,
      cacheTime: 10 * 60_000,
      retry: (count, err) => (isClientError(err) ? false : count < 3),
      refetchOnWindowFocus: false, // search app: avoid surprise refetch storms
    },
  },
})
```

Retry only network/5xx, never 4xx validation errors.

## 4. Mutations with rollback

```tsx
const update = useMutation({
  mutationFn: patchDoc,
  onMutate: async (next) => {
    await client.cancelQueries({ queryKey: docKeys.detail(next.id) })
    const prev = client.getQueryData(docKeys.detail(next.id))
    client.setQueryData(docKeys.detail(next.id), next) // optimistic
    return { prev }
  },
  onError: (_e, vars, ctx) => {
    if (ctx?.prev) client.setQueryData(docKeys.detail(vars.id), ctx.prev)
  },
  onSettled: (_d, _e, vars) => {
    client.invalidateQueries({ queryKey: docKeys.detail(vars.id) }) // settled, not success
  },
})
```

Invalidate in `onSettled` so cache also heals after errors.

## 5. Infinite list (search results)

```tsx
useInfiniteQuery({
  queryKey: docKeys.list(filters),
  queryFn: ({ pageParam = 1 }) => fetchDocs({ ...filters, page: pageParam }),
  getNextPageParam: (last) => (last.hasMore ? last.page + 1 : undefined),
})

const flat = data?.pages.flatMap((p) => p.items) ?? []
```

Render from flattened pages; handle `isFetchingNextPage` separately from `isLoading`.

## 6. Devtools

```tsx
{
  import.meta.env.DEV && (
    <ReactQueryDevtools initialIsOpen={false} position="bottom-right" />
  )
}
```

## Checklist

- [ ] Keys via factory, arrays only.
- [ ] `cacheTime`/`isLoading`/`keepPreviousData` (v4 names).
- [ ] `enabled` for dependent queries.
- [ ] Mutations: cancel → optimistic → rollback → invalidate on settled.
