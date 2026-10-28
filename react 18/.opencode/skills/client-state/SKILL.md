---
name: client-state
description: Client state with zustand 4.3, immer 10, xstate 4.38 and @xstate/react 3.2. Use when managing UI state, stores, undoable edits, wizards, map tools, or multi-step workflows. Server state stays in TanStack Query, not here.
---

# Client State (pinned: zustand@4.3, immer@10, xstate@4.38, @xstate/react@3.2)

Offline-first. Rule #1: server cache (react-query) ≠ client state (this skill). Never duplicate fetched docs into zustand.

## 1. Zustand store (scoped, minimal)

```ts
import { create } from 'zustand'
import { devtools } from 'zustand/middleware'
import { immer } from 'zustand/middleware/immer'

type MapUiState = {
  selectedId: string | null
  panelOpen: boolean
  select: (id: string | null) => void
  togglePanel: () => void
}

export const useMapUi = create<MapUiState>()(
  devtools(
    immer((set) => ({
      selectedId: null,
      panelOpen: false,
      select: (id) => set({ selectedId: id }),
      togglePanel: () =>
        set((s) => {
          s.panelOpen = !s.panelOpen
        }),
    })),
    { name: 'map-ui' }
  )
)
```

Rules: one store per feature concern (map-ui, search-ui), selectors narrow (`useMapUi(s => s.selectedId)`), no whole-store subscriptions in leaves.

## 2. Immer usage

```ts
import { produce } from 'immer'

const next = produce((draft) => {
  draft.filters.topics.push(id)
})(state)
```

Immer already inside zustand via middleware — do not wrap `set()` in extra `produce`. Use standalone `produce` only for non-store reducers/utils.

## 3. XState v4 (NOT v5)

This repo is XState **v4**. v5 APIs (`setup()`, `createActor`, `actor.send`) do NOT exist here.

```ts
import { createMachine, interpret } from 'xstate'

const editorMachine = createMachine({
  id: 'editor',
  initial: 'idle',
  states: {
    idle: { on: { EDIT: 'editing' } },
    editing: { on: { SAVE: 'saving', CANCEL: 'idle' } },
    saving: {
      invoke: { src: 'saveDoc', onDone: 'idle', onError: 'editing' },
    },
  },
})
```

React binding is `@xstate/react@3`:

```tsx
import { useMachine } from '@xstate/react'

const [state, send] = useMachine(editorMachine, { services: { saveDoc } })

state.matches('editing') // v4 way
send({ type: 'SAVE' })
```

## 4. When to use what

| Need                                                                   | Tool                         |
| ---------------------------------------------------------------------- | ---------------------------- |
| Toggle, selection, panel, filters (UI)                                 | zustand                      |
| Nested draft editing with undo steps                                   | zustand + immer              |
| Wizard / lifecycle with forbidden transitions (draft→review→published) | xstate v4                    |
| Fetched lists, pagination, cache                                       | TanStack Query (other skill) |

Do not model a 2-boolean toggle as a state machine; do not model a 6-step approval flow as loose booleans.

## 5. Reset on unmount / scope switch

```ts
reset: () => set({ selectedId: null, panelOpen: false }),
```

Call `reset()` when leaving the feature (route change, doc switch) — stale selection is the #1 state bug.

## Checklist

- [ ] No server data duplicated in zustand.
- [ ] Narrow selectors, feature-scoped stores.
- [ ] XState v4 API only (`matches`, `send({type})`, `interpret`).
- [ ] Reset path exists for feature state.
