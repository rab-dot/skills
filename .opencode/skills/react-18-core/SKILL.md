---
name: react-18-core
description: React 18.2 + ReactDOM + react-error-boundary patterns. Use when writing React components, hooks, memoization, effects, context, portals, error boundaries, or optimizing re-renders in this repo. Do NOT apply React 19 APIs (no Server Components, Actions, use() hook).
---

# React 18 Core (pinned: react@18.2, react-dom@18.2, react-error-boundary@4)

Offline-first. All rules below apply to React 18 in this repo.

## 1. Root API (React 18 only)

```tsx
// main.tsx — correct for 18
import { createRoot } from 'react-dom/client'
import App from './App'

createRoot(document.getElementById('root')!).render(<App />)
```

Never use legacy `ReactDOM.render` (removed in 18 types) and never use `use()` / async Server Components (React 19 only).

## 2. Concurrent features available in 18

- `useTransition` for non-blocking state updates (search, tabs, filters).
- `useDeferredValue` for derived expensive values.
- `useId` for a11y ids (do NOT use for list keys).
- `useSyncExternalStore` for external store subscriptions (zustand backing, manual stores).
- Automatic batching is ON everywhere (including timeouts/promises) — do not micro-optimize around it.

```tsx
const [isPending, startTransition] = useTransition()
const deferredQuery = useDeferredValue(query)

function onSearch(next: string) {
  startTransition(() => setQuery(next)) // heavy list update stays non-blocking
}
```

## 3. State rules

1. Derive during render; do not mirror props into state + effect.
2. Functional updates when next state depends on previous (`setList(prev => [...prev, item])`).
3. Lazy init for expensive initial state: `useState(() => buildHeavy())`.
4. `useReducer` for state with 3+ related fields or transitions (form wizards, map tools).
5. Split contexts by update frequency (theme+user in one context = re-render cascade).

```tsx
// BAD: effect-derived state
const [filtered, setFiltered] = useState(items)
useEffect(() => {
  setFiltered(items.filter(f))
}, [items, f])

// GOOD: derive during render
const filtered = useMemo(() => items.filter(f), [items, f])
```

## 4. Memoization (measure first)

- `React.memo` only for expensive pure leaves with stable props.
- `useMemo` for expensive calculations (>1ms or large lists), not for every object literal.
- `useCallback` only when the function is a dep of a memoized child or effect.

```tsx
const rows = useMemo(() => buildRows(data), [data])
const onSelect = useCallback((id: string) => select(id), [select])
```

## 5. Effects

1. No effects for derived state or pure data transforms.
2. Always clean up subscriptions/timers/listeners (Leaflet layers, SignalR, Howler, map events).
3. Never put fresh objects/arrays in dep arrays — memoize or split into primitives.
4. Effects must be concurrent-safe: no unguarded mutations during render.

```tsx
useEffect(() => {
  const sub = store.subscribe(handler)
  return () => sub.unsubscribe() // cleanup is mandatory
}, [handler])
```

## 6. Error boundaries (react-error-boundary@4)

```tsx
import { ErrorBoundary } from 'react-error-boundary'

;<ErrorBoundary
  fallbackRender={({ error, resetErrorBoundary }) => (
    <div role="alert">
      <p>Something broke: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  )}
  onReset={() => resetQuery()}
>
  <Feature />
</ErrorBoundary>
```

Place boundaries per feature/widget, not one global wrapper — a map crash must not kill the whole page.

## 7. Portals, refs, keys

- Portals for modals/tooltips over Leaflet/Mapbox canvases (`createPortal`).
- `key` to reset state when switching entities: `<Editor key={docId} />`.
- Controlled vs uncontrolled: pick one per input; do not mix `value` + `defaultValue` updates.

## 8. StrictMode double-effects

Dev `StrictMode` mounts effects twice. All subscriptions (SignalR, map listeners, audio) must be idempotent via cleanup. If a bug appears only in dev, check missing cleanup first.

## Checklist before finishing

- [ ] No React 19 API used.
- [ ] No derived state in effects.
- [ ] Every subscription/timer has cleanup.
- [ ] Error boundary at feature level.
- [ ] Memoization justified, not blanket.
