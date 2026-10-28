---
name: react-router-v6
description: React Router v6.8 routing. Use when adding routes, nested layouts, params, search params, navigation, guards, or code-splitting by route.
---

# React Router v6 (pinned: react-router-dom@6.8)

Offline-first. Covers only the v6 API used in this repo.

## 1. Router setup

```tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <RouteError />,
    children: [
      { index: true, element: <SearchPage /> },
      {
        path: 'docs/:id',
        element: <DocPage />,
        loader: docLoader, // fetch before render
      },
    ],
  },
])

export function App() {
  return <RouterProvider router={router} />
}
```

Prefer `createBrowserRouter` (data router) over `<BrowserRouter>` + `<Routes>` for new code: loaders + error boundaries come free.

## 2. Data loading with loaders (avoid waterfalls)

```tsx
import { defer, useLoaderData, Await } from 'react-router-dom'

export async function docLoader({ params }) {
  const doc = await fetchDoc(params.id) // critical
  const related = fetchRelated(params.id) // non-critical -> defer
  return defer({ doc, related })
}
```

Render non-critical via `<Suspense>` + `<Await>` so the page paints fast.

## 3. Params and search params

```tsx
import { useParams, useSearchParams } from 'react-router-dom'

const { id } = useParams() // /docs/:id
const [searchParams, setSearchParams] = useSearchParams()
const page = Number(searchParams.get('page') ?? '1')

function setPage(next: number) {
  setSearchParams((prev) => ({
    ...Object.fromEntries(prev),
    page: String(next),
  }))
}
```

Keep filter state (search, page, sort) in search params — deep-linkable and back-button friendly.

## 4. Navigation

```tsx
import { useNavigate, Link, NavLink } from 'react-router-dom'

const navigate = useNavigate()
navigate(`/docs/${id}`) // push
navigate(-1) // back
navigate('.', { replace: true }) // replace without history entry
```

- `<Link>` for plain navigation, `<NavLink>` for active styling.
- Relative navigation (`navigate('..')`) inside nested routes.

## 5. Guards and auth redirects

```tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom'

export function RequireAuth() {
  const location = useLocation()
  if (!isAuthenticated()) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }
  return <Outlet />
}
```

Compose guards as layout routes, never inside page components with ad-hoc redirects.

## 6. Code splitting per route

```tsx
import { lazy, Suspense } from 'react';

const MapPage = lazy(() => import('./pages/MapPage'));

{ path: 'map', element: <Suspense fallback={<PageSkeleton />}><MapPage /></Suspense> }
```

Heavy routes (map with Leaflet/Mapbox, Excel import) must be lazy — never in the main bundle.

## 7. Common mistakes

1. `useNavigate` outside `<RouterProvider>` — always render inside router context (tests need `MemoryRouter`).
2. Catch-all `*` route for 404 must be last sibling.
3. Do not parse `window.location` manually — use `useSearchParams`/`useParams`.
4. Scroll restoration: add `ScrollRestoration` from `react-router-dom` once in root layout.

## Checklist

- [ ] Routes declared as data router with `errorElement`.
- [ ] Filters in search params, not local-only state.
- [ ] Heavy pages lazy-loaded.
- [ ] Auth guard as layout route.
