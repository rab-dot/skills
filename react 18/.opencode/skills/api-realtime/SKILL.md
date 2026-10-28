---
name: api-realtime
description: HTTP and realtime with axios 1.3 and @microsoft/signalr 7.0. Use when calling REST APIs, configuring interceptors, handling auth tokens and errors, or wiring SignalR hubs for live updates.
---

# API + Realtime (pinned: axios@1.3, @microsoft/signalr@7.0)

Offline-first. One axios instance, one hub manager — no ad-hoc `fetch` scattered in components.

## 1. Axios instance

```ts
import axios, { AxiosError } from 'axios'

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_HOST,
  timeout: 15_000,
  withCredentials: false,
})

api.interceptors.request.use((config) => {
  const token = getToken()
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

api.interceptors.response.use(
  (r) => r,
  (error: AxiosError<ApiError>) => {
    if (error.response?.status === 401) logoutAndRedirect()
    return Promise.reject(normalizeError(error))
  }
)
```

Timeouts mandatory (maps/Excel endpoints hang); 401 handled once globally, not per call.

## 2. Typed calls + error narrowing

```ts
export async function fetchDoc(id: string): Promise<Doc> {
  const { data } = await api.get<Doc>(`/docs/${id}`)
  return data
}

function isClientError(e: unknown): boolean {
  return (
    axios.isAxiosError(e) &&
    !!e.response &&
    e.response.status >= 400 &&
    e.response.status < 500
  )
}
```

`axios.isAxiosError` is the only safe narrowing. Retry policy (react-query skill): retry network/5xx, never 4xx.

## 3. Query integration (no fetching in components)

Components call hooks (TanStack Query v4), hooks call `api.*` functions. No `api.get` inside `useEffect`.

```ts
export function useDoc(id: string) {
  return useQuery({
    queryKey: docKeys.detail(id),
    queryFn: () => fetchDoc(id),
    enabled: !!id,
  })
}
```

## 4. SignalR v7 hub lifecycle

```ts
import * as signalR from '@microsoft/signalr'

let connection: signalR.HubConnection | null = null

export async function connectHub(
  token: string
): Promise<signalR.HubConnection> {
  if (connection?.state === signalR.HubConnectionState.Connected)
    return connection
  connection = new signalR.HubConnectionBuilder()
    .withUrl(`${import.meta.env.VITE_API_HOST}/hubs/docs`, {
      accessTokenFactory: () => token,
    })
    .withAutomaticReconnect([0, 2000, 10_000, 30_000])
    .configureLogging(signalR.LogLevel.Warning)
    .build()
  await connection.start()
  return connection
}

export async function disconnectHub(): Promise<void> {
  await connection?.stop()
  connection = null
}
```

Subscribe/unsubscribe symmetrically, cleanup in effect (StrictMode double-mount safe):

```tsx
useEffect(() => {
  let off = false
  connectHub(token).then((c) => {
    if (off) return
    const handler = (msg: DocEvent) => applyEvent(msg)
    c.on('DocChanged', handler)
    cleanup.current = () => c.off('DocChanged', handler)
  })
  return () => {
    off = true
    cleanup.current?.()
  }
}, [token])
```

Apply incoming events into react-query cache (`setQueryData`/`invalidateQueries`), never into parallel local copies.

## 5. Uploads (ties into files-excel skill)

Use `onUploadProgress` for progress bars; large Excel imports go through presigned upload + job polling, not one giant blocking POST.

## Checklist

- [ ] Single `api` instance with auth + 401 + timeout.
- [ ] No raw fetch in components.
- [ ] SignalR reconnect policy + symmetric on/off + cleanup.
- [ ] Hub events write into query cache.
