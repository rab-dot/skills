---
name: scrollytelling-motion
description: >-
  Scroll-driven storytelling and immersive cinematic transitions with Framer Motion 11 and CSS sticky:
  Sticky section pinning (Apple-style product assemblies), scroll-linked scrubbing, text reveal on scroll,
  multi-layer parallax, and Lenis smooth scrolling integration.
  APPLY ONLY to: marketing landing pages, product launch narratives, interactive case studies, and feature walk-throughs.
  STRICTLY FORBIDDEN in: transactional workflows, checkout pages, data tables, and settings.
---

# Scrollytelling & Scroll-Driven Motion (Framer Motion 11)

Transform passive page scrolling into an interactive, cinematic presentation where the user's
scroll position directly scrub-controls animations, camera pans, and UI assembly.

---

## 1. Core Mechanics & Guardrails

1. **Scroll is user agency**: Never hijack or lock the user's scroll without an obvious escape path.
2. **Scrubbing vs Triggers**:
   - *Scrubbing* (`useTransform`): animation progress matches exact pixel scroll.
   - *Triggering* (`whileInView`): animation fires once when element enters viewport.
3. **No heavy paints on scroll**: NEVER scrub `filter: blur()`, `box-shadow`, or dimensions (`width`/`height`). Scrub ONLY `transform` (`x`, `y`, `scale`, `rotate`) and `opacity`.

---

## 2. Sticky Section Pinning (Apple-Style Presentation)

The classic Apple pattern: the viewport freezes on a sticky container while 3 steps of a feature animate across 300vh of scroll distance.

```tsx
import React, { useRef } from 'react'
import { motion, useScroll, useTransform } from 'framer-motion'

export function StickyFeatureStory() {
  const containerRef = useRef<HTMLDivElement>(null)

  // Track scroll progress across the 300vh container (0.0 to 1.0)
  const { scrollYProgress } = useScroll({
    target: containerRef,
    offset: ['start start', 'end end'],
  })

  // Phase 1: Card 1 fades and shrinks out
  const scale1 = useTransform(scrollYProgress, [0, 0.3], [1, 0.85])
  const opacity1 = useTransform(scrollYProgress, [0.2, 0.35], [1, 0])

  // Phase 2: Card 2 zooms in and fades
  const scale2 = useTransform(scrollYProgress, [0.35, 0.65], [0.85, 1])
  const opacity2 = useTransform(scrollYProgress, [0.3, 0.45, 0.6, 0.7], [0, 1, 1, 0])

  // Phase 3: Final state
  const scale3 = useTransform(scrollYProgress, [0.7, 1], [0.85, 1])
  const opacity3 = useTransform(scrollYProgress, [0.65, 0.8], [0, 1])

  return (
    // Outer scroll container (3x viewport height)
    <div ref={containerRef} className="relative h-[300vh] bg-black text-white">
      {/* Sticky viewport frame (stays pinned to screen) */}
      <div className="sticky top-0 h-screen w-full flex items-center justify-center overflow-hidden">
        {/* Step 1 */}
        <motion.div
          style={{ scale: scale1, opacity: opacity1 }}
          className="absolute max-w-xl text-center"
        >
          <span className="text-indigo-400 font-semibold tracking-wider uppercase text-sm">Step 01</span>
          <h2 className="text-5xl font-bold mt-2">Engineered from the inside out</h2>
          <p className="text-gray-400 mt-4 text-lg">Every component designed for sub-millisecond precision.</p>
        </motion.div>

        {/* Step 2 */}
        <motion.div
          style={{ scale: scale2, opacity: opacity2 }}
          className="absolute max-w-xl text-center"
        >
          <span className="text-cyan-400 font-semibold tracking-wider uppercase text-sm">Step 02</span>
          <h2 className="text-5xl font-bold mt-2">Zero-latency Edge Sync</h2>
          <p className="text-gray-400 mt-4 text-lg">State propagated to all edge locations instantly.</p>
        </motion.div>

        {/* Step 3 */}
        <motion.div
          style={{ scale: scale3, opacity: opacity3 }}
          className="absolute max-w-xl text-center"
        >
          <span className="text-emerald-400 font-semibold tracking-wider uppercase text-sm">Step 03</span>
          <h2 className="text-5xl font-bold mt-2">Unbreakable reliability</h2>
          <p className="text-gray-400 mt-4 text-lg">Guaranteed data persistence with cryptographic proofs.</p>
        </motion.div>
      </div>
    </div>
  )
}
```

---

## 3. Text Reveal Scrub (Word by Word)

Text begins dimmed and illuminates word by word as the user scrolls through the section.

```tsx
import React, { useRef } from 'react'
import { motion, useScroll, useTransform, MotionValue } from 'framer-motion'

interface TextRevealProps {
  text: string
  className?: string
}

export function TextReveal({ text, className = '' }: TextRevealProps) {
  const containerRef = useRef<HTMLDivElement>(null)
  const { scrollYProgress } = useScroll({
    target: containerRef,
    offset: ['start 0.8', 'end 0.2'],
  })

  const words = text.split(' ')

  return (
    <div ref={containerRef} className={`relative min-h-[120vh] py-24 ${className}`}>
      <div className="sticky top-1/3 flex flex-wrap max-w-4xl mx-auto px-6 text-3xl sm:text-5xl font-medium tracking-tight">
        {words.map((word, index) => {
          const start = index / words.length
          const end = start + 1 / words.length
          return (
            <Word key={index} progress={scrollYProgress} range={[start, end]}>
              {word}
            </Word>
          )
        })}
      </div>
    </div>
  )
}

function Word({
  children,
  progress,
  range,
}: {
  children: string
  progress: MotionValue<number>
  range: [number, number]
}) {
  const opacity = useTransform(progress, range, [0.15, 1])

  return (
    <span className="relative mr-3 mt-2">
      <motion.span style={{ opacity }} className="text-gray-900 dark:text-white">
        {children}
      </motion.span>
    </span>
  )
}
```

---

## 4. Multi-Layer Parallax

Creating real optical depth with distinct background, midground, and foreground velocities.

```tsx
import React, { useRef } from 'react'
import { motion, useScroll, useTransform } from 'framer-motion'

export function ParallaxHero() {
  const containerRef = useRef<HTMLDivElement>(null)
  const { scrollYProgress } = useScroll({
    target: containerRef,
    offset: ['start start', 'end start'],
  })

  // Background moves slowest (deep distance)
  const yBg = useTransform(scrollYProgress, [0, 1], ['0%', '20%'])
  // Content moves at normal speed
  const yContent = useTransform(scrollYProgress, [0, 1], ['0%', '50%'])
  // Foreground decorative elements move fastest (closest to viewer)
  const yForeground = useTransform(scrollYProgress, [0, 1], ['0%', '80%'])

  return (
    <div
      ref={containerRef}
      className="relative h-[110vh] overflow-hidden bg-slate-900 flex items-center justify-center"
    >
      {/* Background layer */}
      <motion.div
        style={{ y: yBg }}
        className="absolute inset-0 bg-[radial-gradient(ellipse_at_top,_var(--tw-gradient-stops))] from-indigo-900/40 via-slate-900 to-black pointer-events-none"
      />

      {/* Midground: Hero Text */}
      <motion.div
        style={{ y: yContent }}
        className="relative z-10 text-center max-w-3xl px-6"
      >
        <h1 className="text-6xl font-extrabold text-white tracking-tight">
          Next Generation Architecture
        </h1>
        <p className="mt-6 text-xl text-slate-300">
          Designed for speed, built for security, scale without limits.
        </p>
      </motion.div>

      {/* Foreground accent floating badges */}
      <motion.div
        style={{ y: yForeground }}
        className="absolute bottom-10 left-12 z-20 hidden md:block rounded-xl border border-white/10 bg-white/5 p-4 backdrop-blur-md text-white text-sm"
      >
        ⚡ Global Edge Latency: &lt; 18ms
      </motion.div>
    </div>
  )
}
```

---

## 5. Smooth Scroll Integration (Lenis Standard)

When cinematic momentum scroll is needed, use **Lenis** (industry benchmark, ~3.5 KB, does not break native accessibility or keyboard Tab navigation).

### 5.1 Lenis Hook / Provider Pattern
```tsx
import { useEffect } from 'react'
import Lenis from 'lenis'

export function useSmoothScroll() {
  useEffect(() => {
    // Check if user prefers reduced motion
    const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
    if (prefersReducedMotion) return

    const lenis = new Lenis({
      duration: 1.2,
      easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // exponential ease-out
      orientation: 'vertical',
      gestureOrientation: 'vertical',
      smoothWheel: true,
      wheelMultiplier: 1,
      touchMultiplier: 2,
    })

    function raf(time: number) {
      lenis.raf(time)
      requestAnimationFrame(raf)
    }

    const rafId = requestAnimationFrame(raf)

    return () => {
      cancelAnimationFrame(rafId)
      lenis.destroy()
    }
  }, [])
}
```

---

## Checklist — Scrollytelling Review

- [ ] Sticky containers have a fixed outer height (`h-[200vh]` to `h-[400vh]`)
- [ ] Only `transform` and `opacity` are linked to scroll progress (60+ FPS guarantee)
- [ ] All sticky scroll sections work properly on mobile (test touch scroll velocity)
- [ ] `prefers-reduced-motion` bypasses sticky transforms and renders static stacked cards
- [ ] No horizontal scrollbars triggered unexpectedly by parallax transforms (`overflow-x: hidden`)
