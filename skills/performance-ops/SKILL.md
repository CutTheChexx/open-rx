---
name: performance-ops
description: >
  Web performance optimization knowledge from Google's web.dev, Core Web Vitals best
  practices, and production performance patterns. Use this skill whenever optimizing
  page speed, improving Core Web Vitals (LCP, INP, CLS), reducing bundle size,
  improving load times, or making any web app faster. Triggers on "performance",
  "speed", "optimize", "slow", "Core Web Vitals", "LCP", "lighthouse", "bundle size",
  "lazy load", "code splitting", or any request to make something faster.
metadata:
  version: "1.0.0"
  sources: "web.dev, Core Web Vitals, Lighthouse, Next.js performance docs"
author: "CutTheChexx"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Performance Ops — Speed Intelligence

## Core Web Vitals Targets (2026)

| Metric | Good | Needs Work | Poor | What It Measures |
|--------|------|-----------|------|------------------|
| **LCP** | ≤2.5s | ≤4.0s | >4.0s | Largest visible content painted |
| **INP** | ≤200ms | ≤500ms | >500ms | Responsiveness to user input |
| **CLS** | <0.1 | <0.25 | ≥0.25 | Visual stability (layout shifts) |

Only 47% of websites pass all three. Getting all green = competitive advantage.

## LCP Optimization (Largest Contentful Paint)

### What counts as LCP
- `<img>` elements
- `<video>` poster images
- Elements with CSS `background-image`
- Block-level text elements

### Fixes (in priority order)
1. **Preload the LCP image:**
   ```html
   <link rel="preload" as="image" href="/hero.webp" fetchpriority="high">
   ```
2. **Use modern formats:** WebP (26% smaller) or AVIF (50% smaller than JPEG)
3. **Serve correct sizes:**
   ```html
   <img srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1200.webp 1200w"
        sizes="(max-width: 768px) 100vw, 50vw"
        src="hero-800.webp" alt="..." loading="eager" fetchpriority="high">
   ```
4. **Remove render-blocking resources:**
   ```html
   <link rel="stylesheet" href="critical.css">
   <link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">
   ```
5. **Inline critical CSS** — First 14KB of HTML should contain above-fold styles
6. **Use CDN** — Reduce server response time (TTFB target: <800ms)

## INP Optimization (Interaction to Next Paint)

### Key strategies
1. **Break up long tasks** — No JS task should block >50ms
   ```javascript
   // Bad — blocks for 200ms
   items.forEach(item => heavyProcess(item));

   // Good — yields to browser
   async function processItems(items) {
     for (const item of items) {
       heavyProcess(item);
       await new Promise(r => setTimeout(r, 0)); // yield
     }
   }
   ```
2. **Debounce input handlers** — Don't run on every keystroke
3. **Use `startTransition`** for non-urgent React updates
4. **Web Workers** for heavy computation off the main thread

## CLS Optimization (Cumulative Layout Shift)

### Prevention rules
1. **Always set dimensions on images/video:**
   ```html
   <img width="800" height="600" src="..." alt="..." />
   ```
   Or use aspect-ratio:
   ```css
   img { aspect-ratio: 4/3; width: 100%; height: auto; }
   ```
2. **Reserve space for dynamic content** (ads, embeds, lazy-loaded)
3. **Use `transform` for animations** — Never animate `top/left/width/height`
4. **Preload fonts:**
   ```html
   <link rel="preload" as="font" href="/fonts/inter.woff2" type="font/woff2" crossorigin>
   ```
   And use `font-display: swap` or `font-display: optional`

## Bundle Optimization

### Code splitting
```tsx
// Route-level splitting (React.lazy)
const Dashboard = React.lazy(() => import('./features/dashboard'));
const Settings = React.lazy(() => import('./features/settings'));

// With Suspense
<Suspense fallback={<PageSkeleton />}>
  <Dashboard />
</Suspense>
```

### Tree shaking
```javascript
// Bad — imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// Good — imports only what's needed
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

### Bundle analysis targets
| Category | Budget |
|----------|--------|
| Total JS (compressed) | <200KB |
| First-party JS | <100KB |
| Third-party JS | <100KB |
| CSS (compressed) | <50KB |
| Fonts | <100KB (2-3 weights max) |

## Image Optimization Checklist

- [ ] All images in WebP or AVIF format
- [ ] Responsive `srcset` with appropriate breakpoints
- [ ] `loading="lazy"` on below-fold images
- [ ] `loading="eager"` + `fetchpriority="high"` on LCP image
- [ ] `width` and `height` attributes set (prevents CLS)
- [ ] Compressed (target: <100KB for hero, <50KB for thumbnails)
- [ ] CDN delivery with cache headers

## Font Optimization

```css
/* Optimal font loading */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-weight: 100 900;
  font-display: swap; /* Shows fallback immediately, swaps when loaded */
  unicode-range: U+0000-00FF; /* Latin only — reduce file size */
}
```

**Font budget:** 2-3 weights maximum. Variable fonts save multiple file loads.

## React-Specific Performance

```tsx
// Memoize expensive computations
const sortedItems = useMemo(() =>
  items.sort((a, b) => b.score - a.score),
  [items]
);

// Memoize callbacks passed to children
const handleClick = useCallback((id: string) => {
  setSelected(id);
}, []);

// Virtualize long lists (react-window or @tanstack/react-virtual)
import { useVirtualizer } from '@tanstack/react-virtual';
```

## Quick Wins Checklist

- [ ] Preload LCP element
- [ ] Inline critical CSS (<14KB)
- [ ] Defer non-critical JS (`<script defer>`)
- [ ] Compress with gzip/brotli
- [ ] Set image dimensions
- [ ] Use `font-display: swap`
- [ ] Lazy load below-fold images
- [ ] Code split by route
- [ ] Use CDN for static assets
- [ ] Cache with `Cache-Control: max-age=31536000, immutable` for hashed assets

<sub>Open RX by CutTheChexx — The Prescription.</sub>
