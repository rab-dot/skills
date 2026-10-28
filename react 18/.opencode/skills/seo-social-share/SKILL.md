---
name: seo-social-share
description: >-
  Search Engine Optimization (SEO), OpenGraph, Twitter Cards, Schema.org JSON-LD structured data,
  and social preview sharing. Activate when building landing pages, blog/article templates,
  configuring meta tags, or ensuring links look premium when shared across Telegram, Twitter,
  Discord, LinkedIn, and WhatsApp.
---

# SEO & Social Sharing — Premium Link Previews & Discoverability

A top-tier website must look just as polished when its link is pasted into Telegram, Slack,
Twitter, or Discord as it does in the browser. Furthermore, search engines require clean
structured data to render rich snippets (ratings, prices, FAQs).

---

## 1. Social Share Standard (OpenGraph & Twitter Cards)

Every public page must have rich preview tags configured.

### 1.1 Essential `<head>` Meta Tags
```html
<!-- Primary Meta Tags -->
<title>Product Name — Superhuman Speed for Engineering Teams</title>
<meta name="title" content="Product Name — Superhuman Speed for Engineering Teams" />
<meta name="description" content="Streamline development with zero-latency synchronization, end-to-end encryption, and automated workflow triggers." />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<link rel="canonical" href="https://example.com/product" />

<!-- Open Graph / Facebook / Telegram / Discord -->
<meta property="og:type" content="website" />
<meta property="og:url" content="https://example.com/product" />
<meta property="og:title" content="Product Name — Superhuman Speed for Engineering Teams" />
<meta property="og:description" content="Streamline development with zero-latency synchronization, end-to-end encryption, and automated workflow triggers." />
<meta property="og:image" content="https://example.com/og-image.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:site_name" content="Product Name" />

<!-- Twitter / X -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:url" content="https://example.com/product" />
<meta name="twitter:title" content="Product Name — Superhuman Speed for Engineering Teams" />
<meta name="twitter:description" content="Streamline development with zero-latency synchronization, end-to-end encryption, and automated workflow triggers." />
<meta name="twitter:image" content="https://example.com/og-image.png" />
<meta name="twitter:creator" content="@brandhandle" />
```

### 1.2 Image Specification for OG Images
- **Dimensions**: Exactly `1200 × 630 px` (Aspect ratio `1.91:1`).
- **File size**: Under `300 KB` (optimized WebP or PNG).
- **Safe zone**: Keep critical text and logos within the central `1000 × 500 px` box (some apps crop edges into a square or rounded rectangle).
- **Absolute URLs**: Always use absolute `https://...` URLs. Relative paths (`/og.png`) fail in most social crawlers.

---

## 2. Dynamic SEO in React 18 SPA

For single-page applications without server-side rendering, update document metadata dynamically on route changes:

```tsx
import { useEffect } from 'react'

interface SeoProps {
  title: string
  description: string
  canonicalUrl?: string
  ogImage?: string
}

export function SeoHead({
  title,
  description,
  canonicalUrl,
  ogImage = 'https://example.com/default-og.png',
}: SeoProps) {
  useEffect(() => {
    // 1. Title
    document.title = title

    // 2. Helper to set/create meta tag
    const setMeta = (name: string, content: string, isProperty = false) => {
      const selector = isProperty ? `meta[property="${name}"]` : `meta[name="${name}"]`
      let el = document.querySelector(selector)
      if (!el) {
        el = document.createElement('meta')
        if (isProperty) el.setAttribute('property', name)
        else el.setAttribute('name', name)
        document.head.appendChild(el)
      }
      el.setAttribute('content', content)
    }

    setMeta('description', description)
    setMeta('og:title', title, true)
    setMeta('og:description', description, true)
    setMeta('og:image', ogImage, true)
    setMeta('twitter:title', title)
    setMeta('twitter:description', description)
    setMeta('twitter:image', ogImage)

    // 3. Canonical link
    if (canonicalUrl) {
      let link = document.querySelector('link[rel="canonical"]')
      if (!link) {
        link = document.createElement('link')
        link.setAttribute('rel', 'canonical')
        document.head.appendChild(link)
      }
      link.setAttribute('href', canonicalUrl)
    }
  }, [title, description, canonicalUrl, ogImage])

  return null
}
```

---

## 3. Schema.org (JSON-LD) Structured Data

Rich Google search results (breadcrumbs, FAQ accordions, product pricing, review stars) are powered by JSON-LD blocks in `<head>`.

### 3.1 Organization & Software/Product
```tsx
export function OrganizationSchema() {
  const schemaData = {
    '@context': 'https://schema.org',
    '@type': 'SoftwareApplication',
    name: 'Nexus Cloud',
    applicationCategory: 'DeveloperApplication',
    operatingSystem: 'Web, macOS, Windows, Linux',
    offers: {
      '@type': 'Offer',
      price: '29.00',
      priceCurrency: 'USD',
    },
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: '4.9',
      reviewCount: '1240',
    },
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schemaData) }}
    />
  )
}
```

### 3.2 FAQ Page Schema (renders native FAQ drops in Google search)
```tsx
export function FaqSchema({ faqs }: { faqs: { question: string; answer: string }[] }) {
  const schemaData = {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map(item => ({
      '@type': 'Question',
      name: item.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: item.answer,
      },
    })),
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schemaData) }}
    />
  )
}
```

---

## 4. Modern Favicon & App Kit

Never use a single blurry 16x16 `.ico`. Modern browsers, mobile bookmarks, and dark-mode tabs require a vector kit:

```html
<!-- SVG Favicon with Dark Mode awareness -->
<link rel="icon" href="/favicon.svg" type="image/svg+xml" />

<!-- Fallback for legacy browsers -->
<link rel="alternate icon" href="/favicon.ico" />

<!-- Apple Touch Icon for iOS home screen -->
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />

<!-- Web App Manifest for PWA installation -->
<link rel="manifest" href="/site.webmanifest" />
<meta name="theme-color" content="#0f172a" media="(prefers-color-scheme: dark)" />
<meta name="theme-color" content="#ffffff" media="(prefers-color-scheme: light)" />
```

### Dynamic Dark/Light Mode in `favicon.svg`:
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <style>
    path { fill: #000000; }
    @media (prefers-color-scheme: dark) {
      path { fill: #ffffff; }
    }
  </style>
  <path d="M50 10 L90 90 L10 90 Z" />
</svg>
```

---

## Checklist — SEO & Social Share Gate

- [ ] Every public route has unique `title` and `description`
- [ ] `og:image` is absolute (`https://`), `1200x630px`, and under 300KB
- [ ] `twitter:card` set to `summary_large_image`
- [ ] Canonical URL link present on all pages
- [ ] FAQ or Product pages include validated JSON-LD schema
- [ ] SVG favicon adapts to system dark/light theme
