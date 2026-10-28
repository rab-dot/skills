---
name: ui-system
description: UI with Chakra UI v2.8, NextUI v2, Emotion, Tailwind v3, react-icons. Use when building layouts, theme, color mode, forms UI, modals, tables markup, or styling. Chakra v2 API only (extendTheme), Tailwind v3 config only.
---

# UI System (pinned: chakra@2.8, nextui@2.4, emotion@11, tailwind@3.3, react-icons@4.7)

Offline-first. Pick ONE primary system per screen (Chakra OR NextUI), never mix both in one component tree branch.

## 1. Version traps

- Chakra **v2**: `extendTheme`, `<ChakraProvider theme={theme}>`, `useColorMode`. v3 (`createSystem`, `<Provider>`) is FORBIDDEN here.
- Tailwind **v3**: `tailwind.config.js` + directives `@tailwind base; @tailwind components; @tailwind utilities;`. v4 (`@import "tailwindcss"`, `@theme`) is FORBIDDEN.
- NextUI **v2** package name is `@nextui-org/react` with `NextUIProvider`.

```tsx
// Chakra v2 theme — correct
import { extendTheme, ChakraProvider } from '@chakra-ui/react'

const theme = extendTheme({
  config: { initialColorMode: 'light', useSystemColorMode: false },
  colors: { brand: { 500: '#1d4ed8' } },
})

;<ChakraProvider theme={theme}>
  <App />
</ChakraProvider>
```

## 2. Layout doctrine

1. Page skeleton: app shell (header/sidebar) in widgets layer, page content in pages layer.
2. Spacing scale only (`4, 6, 8`), no magic px.
3. Dark-mode-safe colors: always pair `light`/`dark` values or semantic tokens, never hardcode `#fff` text.
4. Focus states visible (a11y lint enforced); icon-only buttons always get `aria-label`.

## 3. Tailwind v3 + Chakra coexistence

Tailwind for layout spacing/positioning; Chakra/NextUI components for interactive controls. Do not restyle Chakra internals with `!important` Tailwind overrides — use theme props (`variant`, `colorScheme`, `size`).

```js
// tailwind.config.js (v3) — content globs must cover all layers
module.exports = {
  content: ['./index.html', './src/**/*.{ts,tsx}'],
  theme: { extend: {} },
  plugins: [],
}
```

## 4. Forms UI wiring (with forms-yup skill)

```tsx
import {
  FormControl,
  FormLabel,
  FormErrorMessage,
  Input,
} from '@chakra-ui/react'

;<FormControl isInvalid={!!errors.title}>
  <FormLabel>Title</FormLabel>
  <Input {...register('title')} />
  <FormErrorMessage>{errors.title?.message}</FormErrorMessage>
</FormControl>
```

`isInvalid` + `FormErrorMessage` always paired with RHF errors — no silent failures.

## 5. Modals / drawers

One modal system per flow; Escape + overlay click behavior explicit; focus trap kept (do not disable `trapFocus` unless a map canvas needs it, then document why). Long lists inside modals virtualize (see tanstack-table-virtual).

## 6. i18n-aware components

`@internationalized/date` + `@react-aria/i18n` for calendars/date fields; never hand-roll locale parsing. Display formatting via dates skill (`date-fns-tz`), not inline `toLocaleString` hacks.

## 7. Icons (react-icons@4.7)

```tsx
import { FiSearch, FiMapPin } from 'react-icons/fi'
```

Import from subpackage (`/fi`, `/md`), never from root `react-icons` (bundle killer).

## Checklist

- [ ] One UI system per screen.
- [ ] Chakra v2 + Tailwind v3 syntax only.
- [ ] Errors wired to `FormErrorMessage`.
- [ ] Icon-only buttons labelled; dark mode checked.
