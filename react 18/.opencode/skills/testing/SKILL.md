---
name: testing
description: Tests with vitest 0.30, testing-library react 14 + user-event 14 + jest-dom 5, jsdom 21, msw 1.1 and cypress e2e. Use when writing unit, component, integration or e2e tests, mocks, or MSW handlers.
---

# Testing (pinned: vitest@0.30, jsdom@21, msw@1.1, RTL@14, cypress via e2e script)

Offline-first. Pyramid: many unit, some component/integration, few e2e.

## 1. Commands

- `test: vitest` (watch), `test:ci: vitest --reporter=junit`.
- `e2e: cypress run --browser chrome --headless --reporter junit`.

## 2. Unit (pure logic: turf helpers, yup schemas, formatters)

```ts
import { describe, it, expect } from 'vitest'

describe('bbox filter', () => {
  it('keeps points inside polygon', () => {
    expect(filterPoints(points, poly)).toHaveLength(2)
  })
})
```

No DOM, no mocks — fast. Table-driven via `it.each` for validators.

## 3. Component (RTL@14 + user-event@14)

```tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import '@testing-library/jest-dom'

it('shows field error', async () => {
  const user = userEvent.setup()
  render(<DocForm onSubmit={vi.fn()} />)
  await user.click(screen.getByRole('button', { name: /save/i }))
  expect(await screen.findByText(/title is required/i)).toBeInTheDocument()
})
```

Rules: query by role/label (a11y-correct), `userEvent` over `fireEvent`, `findBy*` for async, wrap router/store providers via custom `renderWithProviders`. MSW v1 for network (see below), never stub `fetch` by hand per test.

## 4. MSW v1 (NOT v2)

This repo is MSW **v1**: `rest` + `setupServer`. v2 (`http`, `HttpResponse`) is FORBIDDEN.

```ts
// src/test/mocks/handlers.ts
import { rest } from 'msw'

export const handlers = [
  rest.get(`${import.meta.env.VITE_API_HOST}/docs/:id`, (req, res, ctx) =>
    res(ctx.status(200), ctx.json({ id: req.params.id, title: 'Test' }))
  ),
]

// setupTests.ts
import { setupServer } from 'msw/node'
const server = setupServer(...handlers)
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }))
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

`onUnhandledRequest: 'error'` catches stray calls; per-test overrides via `server.use(...)`.

## 5. E2E (cypress)

Critical paths only: login → search → open doc → export. Selectors by `data-testid`, never CSS/XPath. Seed via API tasks, clean state per spec. Map specs tolerate tile loading (assert markers/UI, not pixels).

## 6. What not to test

No snapshot of huge tables/maps; no testing library internals (zustand impl, query cache); no real network/tokens in CI.

## Checklist

- [ ] MSW v1 syntax (`rest`, `ctx`) only.
- [ ] Role-based queries + user-event.
- [ ] Handlers reset between tests; unhandled = error.
- [ ] E2E minimal, data-testid selectors.
