---
name: build-tooling
description: Build with vite 4, @vitejs/plugin-react 3 and typescript. Use when configuring vite.config.ts, env vars, aliases, proxies, code splitting, or fixing tsc/vite build errors.
---

# Build Tooling (pinned: vite@4, @vitejs/plugin-react@3, typescript@*)

Offline-first. Reproducible builds: versions pinned, env explicit, aliases single-sourced.

## 1. vite.config.ts baseline

```ts
import { defineConfig, loadEnv } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  return {
    plugins: [react()],
    resolve: { alias: { '@': '/src' } }, // mirror of tsconfig paths
    server: {
      port: 3001,
      proxy: { '/api': { target: env.VITE_API_HOST, changeOrigin: true } },
    },
    build: {
      target: 'esnext',
      outDir: 'dist',
      sourcemap: mode !== 'production',
    },
  }
})
```

Scripts in this template: `dev` (vite), `dev-local` (local script + vite), `dev-server` (mock node server + dev-local), `build: tsc && vite build`, `preview: vite preview`.

## 2. Env contract

```ts
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_HOST: string
  readonly VITE_MOCK_API_HOST?: string
  readonly VITE_APP_VERSION?: string
  readonly VITE_MD5_SALT?: string
  readonly VITE_IS_MOCK?: string
}
```

Access only via `import.meta.env.VITE_*`; validate at startup (fail fast with readable message if `VITE_API_HOST` missing). `VITE_*` is public — no secrets (see security-utils).

## 3. Aliases (single source of truth)

```json
// tsconfig.json
{ "compilerOptions": { "baseUrl": ".", "paths": { "@/*": ["src/*"] } } }
```

`@/*` in tsconfig AND `resolve.alias` in vite config must match. Relative imports only inside one slice; cross-slice via `@/` (FSD rule — see code-quality skill).

## 4. Code splitting

```ts
// heavy map/excel routes lazy (see react-router-v6)
const MapPage = lazy(() => import('@/pages/MapPage'))
```

Manual chunks only when measured: `build.rollupOptions.output.manualChunks` for `leaflet`/`mapbox-gl`/`exceljs` if they bloat the main chunk. Keep main entry lean (<300KB gzip target).

## 5. TypeScript gate

`yarn build` runs `tsc` BEFORE vite — type errors fail the build. Keep `strict: true`, `skipLibCheck: true`, `isolatedModules: true`. Fix order: tsconfig paths → unused imports → null checks → vite-only errors (glob imports, `?raw`).

## Checklist

- [ ] Aliases match in tsconfig + vite.
- [ ] Env typed + validated at startup.
- [ ] Heavy deps behind lazy routes.
- [ ] `tsc` clean before `vite build`.
