---
name: design-to-code
description: >-
  Bridge between design decisions and code implementation — theme configuration,
  design token setup, CSS layout patterns (Grid, Flexbox), responsive implementation,
  image optimization, and perceived performance patterns (skeletons, optimistic updates).
  Activate when implementing visual designs in code: setting up themes, building layouts,
  making things responsive, or optimizing perceived performance.
---

# Design → Code Bridge

This skill translates design principles into concrete code. It connects the design skills
(ui-ux-design, design-psychology, ux-patterns) with the technical skills (ui-system, react-18-core).

**Rule:** Never write CSS or styled components by "feel." Every value must trace back to
a design token or a principle from the design skills.

---

## 1. Theme Configuration — Single Source of Truth

Design tokens from `ui-ux-design` must live in ONE place: the theme config.
Never hardcode colors, spacing, or fonts in components.

### 1.1 Chakra v2 theme setup (complete)

```ts
// src/shared/theme/index.ts
import { extendTheme, type ThemeConfig } from '@chakra-ui/react'
import { colors } from './colors'
import { typography } from './typography'
import { components } from './components'
import { semanticTokens } from './semantic-tokens'

const config: ThemeConfig = {
  initialColorMode: 'light',
  useSystemColorMode: false,
}

export const theme = extendTheme({
  config,
  colors,
  ...typography,
  space: {
    // 4px base grid — matches design skill spacing scale
    '0.5': '0.125rem', // 2px
    '1':   '0.25rem',  // 4px
    '1.5': '0.375rem', // 6px
    '2':   '0.5rem',   // 8px
    '3':   '0.75rem',  // 12px
    '4':   '1rem',     // 16px
    '5':   '1.25rem',  // 20px
    '6':   '1.5rem',   // 24px
    '8':   '2rem',     // 32px
    '10':  '2.5rem',   // 40px
    '12':  '3rem',     // 48px
    '16':  '4rem',     // 64px
    '20':  '5rem',     // 80px
    '24':  '6rem',     // 96px
  },
  radii: {
    sm:   '6px',    // inputs, badges, chips
    md:   '8px',    // buttons, small cards
    lg:   '12px',   // cards, modals
    xl:   '16px',   // large cards, hero sections
    full: '9999px', // pills, avatars
  },
  shadows: {
    sm:  '0 1px 2px 0 rgb(0 0 0 / 0.05)',
    md:  '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
    lg:  '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
    xl:  '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
  },
  semanticTokens,
  components,
})
```

### 1.2 Color palette implementation

```ts
// src/shared/theme/colors.ts
// Generated from design skill palette — consistent saturation across hues

export const colors = {
  brand: {
    50:  '#eff6ff',
    100: '#dbeafe',
    200: '#bfdbfe',
    300: '#93c5fd',
    400: '#60a5fa',
    500: '#3b82f6', // primary
    600: '#2563eb', // primary hover/active
    700: '#1d4ed8',
    800: '#1e40af',
    900: '#1e3a8a',
  },
  // Neutral — NEVER pure black. Use these for all text and borders.
  gray: {
    50:  '#f9fafb',
    100: '#f3f4f6',
    200: '#e5e7eb',
    300: '#d1d5db',
    400: '#9ca3af',
    500: '#6b7280',
    600: '#4b5563',
    700: '#374151',
    800: '#1f2937',
    900: '#111827',  // darkest text — NOT #000
    950: '#030712',  // dark mode bg
  },
  // Semantic colors
  success: {
    50: '#f0fdf4', 100: '#dcfce7', 500: '#22c55e', 600: '#16a34a', 700: '#15803d',
  },
  warning: {
    50: '#fffbeb', 100: '#fef3c7', 500: '#f59e0b', 600: '#d97706', 700: '#b45309',
  },
  error: {
    50: '#fef2f2', 100: '#fee2e2', 500: '#ef4444', 600: '#dc2626', 700: '#b91c1c',
  },
  info: {
    50: '#eff6ff', 100: '#dbeafe', 500: '#3b82f6', 600: '#2563eb', 700: '#1d4ed8',
  },
}
```

### 1.3 Semantic tokens (light/dark auto-switch)

```ts
// src/shared/theme/semantic-tokens.ts
export const semanticTokens = {
  colors: {
    // Surface layers (elevation system for dark mode)
    'surface.page':     { default: 'white',     _dark: 'gray.950' },
    'surface.card':     { default: 'white',     _dark: 'gray.900' },
    'surface.elevated': { default: 'white',     _dark: 'gray.800' },
    'surface.overlay':  { default: 'gray.50',   _dark: 'gray.700' },

    // Text hierarchy
    'text.primary':   { default: 'gray.900', _dark: 'gray.100' },
    'text.secondary': { default: 'gray.600', _dark: 'gray.400' },
    'text.muted':     { default: 'gray.400', _dark: 'gray.500' },
    'text.inverse':   { default: 'white',    _dark: 'gray.900' },

    // Borders
    'border.default': { default: 'gray.200', _dark: 'gray.700' },
    'border.strong':  { default: 'gray.300', _dark: 'gray.600' },
    'border.focus':   { default: 'brand.500', _dark: 'brand.400' },

    // Interactive states
    'interactive.default': { default: 'brand.600', _dark: 'brand.400' },
    'interactive.hover':   { default: 'brand.700', _dark: 'brand.300' },
    'interactive.muted':   { default: 'brand.50',  _dark: 'brand.900' },
  },
}
```

### 1.4 Typography configuration

```ts
// src/shared/theme/typography.ts
export const typography = {
  fonts: {
    heading: `'Inter', -apple-system, BlinkMacSystemFont, sans-serif`,
    body:    `'Inter', -apple-system, BlinkMacSystemFont, sans-serif`,
    mono:    `'JetBrains Mono', 'Fira Code', monospace`,
  },
  fontSizes: {
    xs:   '0.75rem',   // 12px — captions, badges
    sm:   '0.875rem',  // 14px — secondary text, labels
    md:   '1rem',      // 16px — body (NEVER smaller for body)
    lg:   '1.125rem',  // 18px — lead paragraphs
    xl:   '1.25rem',   // 20px — card titles
    '2xl': '1.5rem',   // 24px — section headings
    '3xl': '1.875rem', // 30px — page titles
    '4xl': '2.25rem',  // 36px — hero headings
    '5xl': '3rem',     // 48px — display
  },
  lineHeights: {
    tight:  '1.25',  // headings
    snug:   '1.375', // subheadings
    normal: '1.5',   // body text minimum
    relaxed: '1.625', // comfortable reading
    loose:  '1.75',  // maximum readability
  },
  letterSpacings: {
    tighter: '-0.025em', // large headings
    tight:   '-0.01em',  // medium headings
    normal:  '0',        // body text
    wide:    '0.05em',   // small uppercase labels
    wider:   '0.1em',    // badges, all-caps
  },
}
```

### 1.5 Component style overrides

```ts
// src/shared/theme/components.ts
import { defineStyleConfig } from '@chakra-ui/react'

const Button = defineStyleConfig({
  baseStyle: {
    fontWeight: '500',
    borderRadius: 'md',
    transition: 'all 150ms ease-out',
    _focusVisible: {
      ring: '2px',
      ringColor: 'border.focus',
      ringOffset: '2px',
    },
  },
  sizes: {
    sm: { h: '36px', fontSize: 'sm', px: '3' },
    md: { h: '40px', fontSize: 'md', px: '4' },
    lg: { h: '48px', fontSize: 'md', px: '6' },
  },
  variants: {
    primary: {
      bg: 'interactive.default',
      color: 'text.inverse',
      _hover: { bg: 'interactive.hover' },
    },
    secondary: {
      bg: 'transparent',
      border: '1px solid',
      borderColor: 'border.default',
      color: 'text.primary',
      _hover: { bg: 'surface.overlay' },
    },
    ghost: {
      bg: 'transparent',
      color: 'text.secondary',
      _hover: { bg: 'surface.overlay', color: 'text.primary' },
    },
    destructive: {
      bg: 'error.600',
      color: 'white',
      _hover: { bg: 'error.700' },
    },
  },
  defaultProps: {
    variant: 'primary',
    size: 'md',
  },
})

const Input = defineStyleConfig({
  baseStyle: {
    field: {
      borderRadius: 'sm',
      transition: 'all 150ms ease-out',
      _focusVisible: {
        borderColor: 'border.focus',
        boxShadow: '0 0 0 1px var(--chakra-colors-brand-500)',
      },
    },
  },
  sizes: {
    md: {
      field: { h: '40px', fontSize: 'md', px: '3' },
    },
  },
})

export const components = { Button, Input }
```

---

## 2. CSS Layout Patterns

### 2.1 Page shell (app layout)

```tsx
// Sidebar + main content — the most common SaaS layout
<div className="flex h-screen overflow-hidden">
  {/* Sidebar — fixed width, scrollable */}
  <aside className="w-64 shrink-0 border-r border-gray-200 overflow-y-auto">
    <nav>{/* sidebar nav */}</nav>
  </aside>

  {/* Main area — fills remaining space */}
  <div className="flex-1 flex flex-col overflow-hidden">
    {/* Top header bar */}
    <header className="h-16 shrink-0 border-b border-gray-200 flex items-center px-6">
      {/* header content */}
    </header>

    {/* Scrollable content area */}
    <main className="flex-1 overflow-y-auto p-6">
      <div className="max-w-7xl mx-auto">
        {/* page content — max-width prevents text stretching on wide screens */}
      </div>
    </main>
  </div>
</div>
```

### 2.2 Card grid (responsive)

```tsx
// Responsive grid: 1 col → 2 col → 3 col → 4 col
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  {items.map(item => (
    <Card key={item.id} item={item} />
  ))}
</div>
```

### 2.3 KPI row

```tsx
// 4 KPI cards in a row, stacking on mobile
<div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
  <KpiCard label="Revenue" value="$48,200" trend="+12.5%" positive />
  <KpiCard label="Users" value="1,240" trend="+8.1%" positive />
  <KpiCard label="Bounce Rate" value="34.2%" trend="-2.3%" positive />
  <KpiCard label="Avg. Session" value="4m 12s" trend="-0.8%" negative />
</div>
```

### 2.4 Content + sidebar layout

```tsx
// Main content with right sidebar (filters, TOC, etc.)
<div className="flex gap-8">
  <div className="flex-1 min-w-0"> {/* min-w-0 prevents flex child overflow */}
    {/* main content */}
  </div>
  <aside className="w-72 shrink-0 hidden lg:block">
    {/* sidebar content — hidden on mobile */}
  </aside>
</div>
```

### 2.5 Centered form / article

```tsx
// Optimal reading width — 65ch ≈ 580px
<div className="max-w-prose mx-auto px-4">
  <h1 className="text-3xl font-bold tracking-tight text-gray-900">
    Page Title
  </h1>
  <p className="mt-4 text-base leading-relaxed text-gray-600">
    Body content with comfortable line-height and readable width.
  </p>
</div>
```

### 2.6 Sticky header + scrollable content

```tsx
// Header stays visible while content scrolls
<div className="flex flex-col h-full">
  <div className="shrink-0 border-b px-6 py-4">
    {/* sticky header area */}
  </div>
  <div className="flex-1 overflow-y-auto px-6 py-4">
    {/* scrollable content */}
  </div>
  <div className="shrink-0 border-t px-6 py-3">
    {/* optional sticky footer — actions bar */}
  </div>
</div>
```

### 2.7 Holy grail layout with CSS Grid

```tsx
<div className="grid grid-cols-[auto_1fr_auto] grid-rows-[auto_1fr_auto] min-h-screen">
  <header className="col-span-3 h-16 border-b">Header</header>
  <aside className="w-64 border-r p-4">Left sidebar</aside>
  <main className="p-6 overflow-y-auto">Main content</main>
  <aside className="w-64 border-l p-4 hidden xl:block">Right sidebar</aside>
  <footer className="col-span-3 h-12 border-t">Footer</footer>
</div>
```

---

## 3. Responsive Implementation

### 3.1 Breakpoint-aware components with Chakra

```tsx
// Responsive props — mobile-first
<Box
  px={{ base: 4, md: 6, lg: 8 }}
  py={{ base: 4, md: 6 }}
  maxW={{ base: '100%', lg: '1280px' }}
  mx="auto"
>
  <Heading
    fontSize={{ base: '2xl', md: '3xl', lg: '4xl' }}
    lineHeight="tight"
    letterSpacing="tighter"
  >
    Page Title
  </Heading>

  <SimpleGrid
    columns={{ base: 1, md: 2, lg: 3 }}
    spacing={{ base: 4, md: 6 }}
    mt={6}
  >
    {items.map(item => <Card key={item.id} {...item} />)}
  </SimpleGrid>
</Box>
```

### 3.2 Hide/show elements responsively

```tsx
// Chakra responsive visibility
<Box display={{ base: 'none', md: 'block' }}>Desktop only</Box>
<Box display={{ base: 'block', md: 'none' }}>Mobile only</Box>

// Tailwind responsive visibility
<div className="hidden md:block">Desktop only</div>
<div className="block md:hidden">Mobile only</div>
```

### 3.3 Responsive navigation pattern

```tsx
// Desktop: sidebar. Mobile: bottom tab bar + hamburger.
function AppShell({ children }: { children: ReactNode }) {
  const isMobile = useBreakpointValue({ base: true, lg: false })

  return (
    <Flex h="100vh" direction={isMobile ? 'column' : 'row'}>
      {!isMobile && <DesktopSidebar />}

      <Box flex="1" overflow="auto">
        {isMobile && <MobileHeader />}
        <Box as="main" p={{ base: 4, lg: 6 }}>
          {children}
        </Box>
      </Box>

      {isMobile && <MobileBottomNav />}
    </Flex>
  )
}
```

### 3.4 Responsive table → card list

```tsx
// Desktop: table. Mobile: card list. Common pattern for data tables.
const isMobile = useBreakpointValue({ base: true, md: false })

{isMobile ? (
  <VStack spacing={3}>
    {data.map(item => (
      <Box key={item.id} p={4} bg="surface.card" borderRadius="lg" shadow="sm" w="full">
        <Text fontWeight="600">{item.name}</Text>
        <HStack mt={2} justify="space-between">
          <Badge colorScheme={statusColor(item.status)}>{item.status}</Badge>
          <Text fontSize="sm" color="text.secondary">{item.date}</Text>
        </HStack>
      </Box>
    ))}
  </VStack>
) : (
  <Table>{/* full table layout */}</Table>
)}
```

---

## 4. Perceived Performance Patterns

### 4.1 Skeleton screens

```tsx
// Skeleton that matches actual content layout — NOT generic spinner
import { Skeleton, SkeletonText, SkeletonCircle } from '@chakra-ui/react'

function UserCardSkeleton() {
  return (
    <Box p={6} bg="surface.card" borderRadius="lg" shadow="sm">
      <HStack spacing={4}>
        <SkeletonCircle size="48px" />
        <Box flex="1">
          <Skeleton h="20px" w="60%" mb={2} />
          <Skeleton h="16px" w="40%" />
        </Box>
      </HStack>
      <SkeletonText mt={4} noOfLines={3} spacing={3} />
    </Box>
  )
}

// Use in suspense or conditional
{isLoading ? (
  <SimpleGrid columns={3} spacing={6}>
    {Array.from({ length: 6 }).map((_, i) => <UserCardSkeleton key={i} />)}
  </SimpleGrid>
) : (
  <SimpleGrid columns={3} spacing={6}>
    {users.map(u => <UserCard key={u.id} user={u} />)}
  </SimpleGrid>
)}
```

### 4.2 Optimistic updates with TanStack Query

```tsx
// Show the result immediately, sync in background
const mutation = useMutation({
  mutationFn: updateTask,
  onMutate: async (updated) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['tasks'] })

    // Snapshot previous value
    const previous = queryClient.getQueryData(['tasks'])

    // Optimistically update cache
    queryClient.setQueryData(['tasks'], (old: Task[]) =>
      old.map(t => t.id === updated.id ? { ...t, ...updated } : t)
    )

    return { previous }
  },
  onError: (_err, _vars, context) => {
    // Rollback on error
    queryClient.setQueryData(['tasks'], context?.previous)
    toast({ title: 'Failed to update', status: 'error' })
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['tasks'] })
  },
})
```

### 4.3 Loading states hierarchy

```tsx
// Follow the Doherty Threshold from design-psychology skill

function DataView() {
  const { data, isLoading, error, isFetching } = useQuery(...)

  // 1. First load — skeleton (shapes match content)
  if (isLoading) return <ContentSkeleton />

  // 2. Error — inline message with retry (not a blank page)
  if (error) return <ErrorState message={error.message} onRetry={refetch} />

  // 3. Empty — helpful empty state with CTA
  if (data.length === 0) return <EmptyState />

  // 4. Content + subtle refetch indicator
  return (
    <>
      {isFetching && <Progress size="xs" isIndeterminate position="sticky" top={0} />}
      <ContentList data={data} />
    </>
  )
}
```

### 4.4 Suspense boundaries (lazy routes)

```tsx
// Per-route suspense with route-specific skeleton
<Suspense fallback={<DashboardSkeleton />}>
  <DashboardPage />
</Suspense>

<Suspense fallback={<TableSkeleton rows={10} cols={5} />}>
  <UsersPage />
</Suspense>
```

---

## 5. Image & Media Optimization

### 5.1 Responsive images

```tsx
// Always specify dimensions to prevent layout shift
<img
  src="/images/hero.webp"
  alt="Dashboard preview"
  width={1200}
  height={675}
  loading="lazy"
  decoding="async"
  className="w-full h-auto rounded-lg"
/>
```

### 5.2 Aspect ratio containers

```tsx
// Consistent card image aspect ratios
<Box position="relative" paddingTop="56.25%" overflow="hidden" borderRadius="lg">
  <Image
    src={imageUrl}
    alt={title}
    position="absolute"
    top={0}
    left={0}
    w="100%"
    h="100%"
    objectFit="cover"
    loading="lazy"
  />
</Box>

// Or with Tailwind — cleaner
<div className="aspect-video overflow-hidden rounded-lg">
  <img src={url} alt={title} className="w-full h-full object-cover" loading="lazy" />
</div>
```

### 5.3 Avatar with fallback

```tsx
<Avatar
  size="md"
  src={user.avatarUrl}
  name={user.displayName}  // Chakra generates initials fallback automatically
  bg="brand.500"
  color="white"
/>
```

---

## 6. Transition & Animation Implementation

### 6.1 Standard interactive transitions (CSS)

```tsx
// Apply to ALL interactive elements in the theme
// Already set in component overrides — but for custom elements:
<Box
  transition="all 150ms ease-out"
  _hover={{ transform: 'translateY(-2px)', shadow: 'lg' }}
  cursor="pointer"
  role="group"
>
  {/* Card content that lifts on hover */}
</Box>
```

### 6.2 Page/section entrance animations (Framer Motion)

```tsx
// Stagger children entrance — for card grids, list items
const container = {
  hidden: {},
  show: {
    transition: { staggerChildren: 0.05 },
  },
}

const item = {
  hidden: { opacity: 0, y: 8 },
  show: { opacity: 1, y: 0, transition: { duration: 0.2 } },
}

<motion.div variants={container} initial="hidden" animate="show">
  {cards.map(card => (
    <motion.div key={card.id} variants={item}>
      <Card {...card} />
    </motion.div>
  ))}
</motion.div>
```

### 6.3 Reduced motion — mandatory

```tsx
// All animation decisions must respect this
import { useReducedMotion } from 'framer-motion'

function AnimatedList({ children }) {
  const shouldReduce = useReducedMotion()

  return shouldReduce ? (
    <div>{children}</div>
  ) : (
    <motion.div variants={container} initial="hidden" animate="show">
      {children}
    </motion.div>
  )
}
```

---

## 7. Focus Management & Keyboard Navigation

### 7.1 Focus-visible styles (in theme)

```tsx
// Global focus style — already in Chakra theme components, but for custom elements:
<button
  className="
    focus:outline-none
    focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2
    rounded-md
  "
>
  Action
</button>
```

### 7.2 Skip link

```tsx
// First focusable element in the DOM — invisible until focused
<a
  href="#main-content"
  className="
    sr-only focus:not-sr-only
    focus:fixed focus:top-4 focus:left-4 focus:z-50
    focus:px-4 focus:py-2 focus:bg-white focus:shadow-lg focus:rounded-md
  "
>
  Skip to main content
</a>

{/* ... navigation ... */}

<main id="main-content" tabIndex={-1}>
  {/* page content */}
</main>
```

### 7.3 Focus trap for modals (Chakra built-in)

```tsx
// Chakra Modal already handles focus trap — but for custom modals:
import { FocusTrap } from 'focus-trap-react'

<FocusTrap active={isOpen}>
  <div role="dialog" aria-modal="true" aria-labelledby="modal-title">
    <h2 id="modal-title">Confirm deletion</h2>
    {/* modal content */}
    <button onClick={onClose}>Cancel</button>
    <button onClick={onConfirm}>Delete</button>
  </div>
</FocusTrap>
```

### 7.4 Keyboard navigation for custom components

```tsx
// Arrow key navigation for a custom list
function useArrowNavigation(itemCount: number) {
  const [activeIndex, setActiveIndex] = useState(0)

  const onKeyDown = useCallback((e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault()
        setActiveIndex(i => Math.min(i + 1, itemCount - 1))
        break
      case 'ArrowUp':
        e.preventDefault()
        setActiveIndex(i => Math.max(i - 1, 0))
        break
      case 'Home':
        e.preventDefault()
        setActiveIndex(0)
        break
      case 'End':
        e.preventDefault()
        setActiveIndex(itemCount - 1)
        break
    }
  }, [itemCount])

  return { activeIndex, onKeyDown }
}
```

---

## Checklist — Design-to-Code Implementation

- [ ] All colors, spacing, fonts come from theme — no hardcoded values in components
- [ ] Semantic tokens handle light/dark mode automatically
- [ ] Layout uses established CSS patterns (flex/grid), not absolute positioning hacks
- [ ] Responsive breakpoints tested: mobile → tablet → desktop
- [ ] Loading states use skeleton screens matching content shape
- [ ] Images have width/height, loading="lazy", and alt text
- [ ] Focus-visible styles on all interactive elements
- [ ] Skip link present as first focusable element
- [ ] Animations respect prefers-reduced-motion
- [ ] Optimistic updates for frequently-used mutations
