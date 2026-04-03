---
name: brand-builder
description: >
  Premium skill for building complete brand identity and design systems for web applications. Covers visual design language, typography systems, color palettes, design tokens, component libraries, brand guidelines, logo specifications, and production-grade design system architecture. Essential for creating cohesive, accessible, scalable visual identities across web products. Handles design system setup, token management, brand voice integration, and design token CSS architecture for light/dark modes.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Design system best practices, brand identity guides, typography resources, accessibility standards"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## Brand Builder: Complete Design System & Brand Identity Toolkit

Professional-grade skill for establishing visual identity, design systems, and brand guidelines for web applications. Production-ready patterns and token architecture.

---

### 1. Color System Architecture

**Building a color palette from scratch:**

- **Primary, Secondary, Accent**: Core brand colors with clear hierarchy. Primary drives CTAs, navigation, key interactions.
- **Semantic Colors**: Success (green), warning (amber), error (red), info (blue). Use consistent lightness ratios across all semantic meanings.
- **Neutral Scale**: 11-step neutral scale from pure white (0) to pure black (11). Building: lightest use #FAFAFA, darkest use #1A1A1A. Middle tones at 50%, 60%, 70%, 80% lightness.
- **HSL-Based System**: Define all colors in HSL for consistency. Hue stays fixed within a color family, adjust saturation and lightness for variations. Example: Primary blue at H:215° S:100% L:50%, with L:95% for backgrounds, L:10% for text.
- **CSS Variable Architecture**: Root variables for base colors. Semantic variables for usage context.

```css
:root {
  --color-primary: hsl(215, 100%, 50%);
  --color-primary-light: hsl(215, 100%, 95%);
  --color-primary-dark: hsl(215, 100%, 25%);
  --color-success: hsl(142, 76%, 36%);
  --color-error: hsl(0, 100%, 50%);
}

[data-theme="dark"] {
  --color-bg-primary: hsl(0, 0%, 10%);
  --color-text-primary: hsl(0, 0%, 95%);
}
```

- **Dark Mode Mapping**: Don't invert colors. Create parallel token sets. Dark backgrounds 10-15% lightness, light text 90-95% lightness. Test WCAG contrast ratios (4.5:1 for text).
- **Accessibility First**: All text must hit 4.5:1 contrast minimum (WCAG AA). Large text (18pt+) needs 3:1 minimum. Use WebAIM contrast checker during palette builds.

---

### 2. Typography System

**Font Selection Strategy:**

- **Google Fonts or Self-Hosted**: Google Fonts provides reliability; self-hosted gives performance control and licensing flexibility. Variable fonts reduce file size significantly.
- **Variable Font Benefits**: Single file replaces 12+ weight/style variants. Include only needed weights (typically 400, 500, 600, 700 for sans-serif).
- **Type Scale (Modular Scale)**: Use 1.125 ratio (minor third) or 1.25 ratio (major third). Starting from 16px body:
  - XS: 14px (body-small, captions)
  - SM: 14px (default body)
  - BASE: 16px (body-default)
  - LG: 18px (body-large, intro text)
  - XL: 20px (heading-6)
  - 2XL: 24px (heading-5)
  - 3XL: 28px (heading-4)
  - 4XL: 32px (heading-3)
  - 5XL: 36px (heading-2)
  - 6XL: 48px (heading-1)

**Hierarchy & Usage:**

- **Headings**: H1 for page titles, H2 for sections, H3 for subsections. Don't skip levels. Always use semantic HTML (h1, h2, etc.).
- **Body Text**: 16px at 1.5 line-height minimum. 14px max for secondary text. Code blocks at monospace, 13-14px.
- **Font Loading Strategy**:
  - Use `font-display: swap` for body fonts (fast render with fallback).
  - Use `font-display: block` for critical fonts (headings).
  - Preload primary font files: `<link rel="preload" as="font" href="font.woff2" type="font/woff2" crossorigin>`
  - Load max 2-3 font families (primary sans + one serif if needed + monospace).

**Fluid Typography (Production Pattern):**

```css
body {
  font-size: clamp(16px, 1.5vw, 18px);
  line-height: 1.6;
}

h1 {
  font-size: clamp(32px, 5vw, 48px);
  line-height: 1.2;
  font-weight: 700;
}
```

Fluid typography scales smoothly between breakpoints without media queries.

---

### 3. Spacing & Layout System

**4px/8px Grid Foundation:**

- **Base Unit**: 4px or 8px. All spacing, padding, margins are multiples. Builds visual harmony.
- **Spacing Scale**:
  - XS: 4px (tight grouping)
  - SM: 8px (component padding)
  - MD: 16px (section spacing)
  - LG: 24px (major section break)
  - XL: 32px (page margins)
  - 2XL: 48px (hero spacing)

**CSS Variables for Spacing:**

```css
:root {
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
}

.card {
  padding: var(--space-md);
  margin-bottom: var(--space-lg);
}
```

**Layout Patterns:**

- **Sidebar Layout**: Fixed sidebar (240-280px) + fluid main content. Use CSS Grid: `grid-template-columns: 280px 1fr`.
- **Dashboard Layout**: Header (60-80px) + sidebar + content. Stack on mobile.
- **Marketing Layout**: Hero (100vh), feature sections (min 400px height), testimonials, CTA footer.
- **Responsive Breakpoints**: Mobile (320-640), Tablet (641-1024), Desktop (1025-1440), Wide (1441+).

---

### 4. Design Tokens & CSS Variables

**Token Naming Convention:**

`--{category}-{modifier}-{state}`

Examples:
- `--color-primary-default`
- `--color-error-light`
- `--space-lg`
- `--border-radius-sm`
- `--shadow-elevation-1`
- `--font-family-body`

**Semantic vs. Primitive:**

- **Primitive**: `--color-blue-500: hsl(215, 100%, 50%)`
- **Semantic**: `--color-button-bg: var(--color-blue-500)` (in light mode), `--color-button-bg: var(--color-blue-600)` (in dark mode)

**Light/Dark Mode Mapping:**

```css
:root {
  --color-bg-primary: #ffffff;
  --color-text-primary: #1a1a1a;
  --color-border: #e0e0e0;
}

[data-theme="dark"] {
  --color-bg-primary: #1a1a1a;
  --color-text-primary: #ffffff;
  --color-border: #404040;
}
```

---

### 5. Component Library Foundation

**Build vs. Use Decision:**

- **Use shadcn/ui** for: buttons, inputs, dialogs, dropdowns, tables, modals. Pre-audited accessibility, excellent defaults.
- **Build Custom** for: brand-specific components (hero, testimonial cards, pricing tables), unique interactions, proprietary patterns.

**Extending shadcn Components:**

```javascript
// Wrap shadcn Button with brand defaults
import { Button as BaseButton } from "@/components/ui/button"

export function Button({ variant = "default", ...props }) {
  return <BaseButton variant={variant} {...props} />
}
```

**Consistent Variant System:**

All components should support: `variant`, `size`, `disabled`, `loading` states. Naming: "primary", "secondary", "ghost", "outline".

**Icon Library:** Use Lucide React (tree-shakeable, consistent design, 1000+ icons). Pair with 24px as default size.

**Border Radius Scale:**
- `--radius-sm: 4px` (inputs, small components)
- `--radius-md: 8px` (cards, buttons, standard)
- `--radius-lg: 12px` (large modals, hero images)
- `--radius-full: 9999px` (circles, pills)

**Shadow System** (elevation):
- SM: `0 1px 2px rgba(0,0,0,0.05)`
- MD: `0 4px 6px rgba(0,0,0,0.1)`
- LG: `0 10px 15px rgba(0,0,0,0.1)`
- XL: `0 20px 25px rgba(0,0,0,0.1)`

---

### 6. Brand Voice in UI

**Copy + Visual Design Unity:**

Brand voice (tone, personality) must reflect in UI:

- **Formal Brand** → Clean typography, spacious layouts, muted colors, minimal iconography.
- **Playful Brand** → Rounded corners, vibrant colors, friendly copy, emoji/illustration use.
- **Minimal Brand** → Thin fonts, high contrast, generous whitespace, icon-first.

**Consistency Touchpoints:**

- Button copy ("Get Started" vs "Start Free Trial" vs "Join Now") — choose one pattern.
- Error messages: prescriptive, never blame user ("Email format required" vs "Invalid email").
- Success feedback: celebrate wins ("Welcome! Let's get started" vs plain "Done").
- Loading states: brief, friendly ("Loading your workspace..." vs "LOADING").

---

### 7. Logo Usage Guidelines

**Logo Variants to Create:**

1. **Full Logo** (horizontal lockup): Primary use case, all contexts.
2. **Icon/Mark**: Standalone symbol, favicons, app icon.
3. **Wordmark**: Logo text only, for tight spaces.
4. **Monochrome**: Single-color version for restricted spaces (t-shirts, stickers).

**Clear Space Rules:**

Define minimum clear space (empty space around logo). Typically 1/3 to 1/2 of logo height.

**Minimum Sizes:**

- Horizontal: 200px width minimum
- Icon: 64px minimum (favicon 32px)
- On dark backgrounds: ensure 4.5:1 contrast

**Do's and Don'ts:**

- Never skew, rotate, or distort
- Never change colors (except monochrome version)
- Never add filters or effects
- Don't place on busy backgrounds without contrast padding
- Don't use in tiny contexts (under 48px, use icon alone)

**Favicon Generation:**

Use favicon.io or realfavicongenerator.net. Generate: favicon-32x32.png, favicon-16x16.png, apple-touch-icon.png. Add to head:

```html
<link rel="icon" type="image/png" href="/favicon-32x32.png" sizes="32x32">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
```

**Open Graph Image:**

Create branded OG image (1200x630px) for social sharing. Template: logo + headline + accent color. Use Figma template or Cloudinary dynamic images.

---

### 8. Brand Guidelines Document Template

**Sections to Include:**

1. **Logo Usage** (variants, clear space, dos/don'ts)
2. **Color Palette** (hex/HSL values, use cases, contrast ratios)
3. **Typography** (font stack, scale, weights, line heights)
4. **Imagery Style** (photography tone, illustration style, filters applied)
5. **Tone of Voice** (3-5 brand personality traits + examples)
6. **Component Showcase** (buttons, cards, forms in brand colors)
7. **UI Patterns** (buttons, CTAs, error states, loading states)
8. **Spacing & Layout** (grid system, margins, padding patterns)
9. **Accessibility** (contrast requirements, icon alt text patterns)
10. **Do's and Don'ts** (visual examples of brand applied correctly vs. incorrectly)

**Maintenance Strategy:**

- Keep guidelines as living document (Figma file or shared notion page).
- Version each update ("v1.0 → v1.1 → v2.0").
- Require review before major brand changes.
- Screenshot key sections as PNG for easy reference.

---

### 9. Design System Maintenance

**Version Control for Design Tokens:**

Track tokens in git. Changelog format:

```
## v1.1.0 (2026-02-15)
- Added new semantic color `--color-info-light`
- Increased `--space-md` from 16px to 18px
- Deprecated `--color-deprecated-teal` (use `--color-secondary` instead)
```

**Component Changelog:**

Document changes per component version. Example: Button v2.0 adds loading state, changes variant names from "primary/secondary" to "default/outline".

**Storybook Setup:**

```bash
npx create-react-app my-design-system
npm install @storybook/react
npx storybook init
```

Create stories for each component with all variants, states. Use Storybook's built-in docs addon to auto-generate component API docs.

**Visual Regression Testing:**

Use Percy.io or Chromatic for automated visual tests. Catches unintended design changes in CI/CD.

---

### 10. Quick-Start Brand Kit (No Brand Provided)

**10-Minute Brand Generation:**

1. **Pick Primary Color** (90 seconds): Use Coolors.co, randomly generate until you love one. Example: `hsl(215, 100%, 50%)` (vibrant blue).
2. **Neutral Scale** (2 min): Use chir.app, generate 11-step grayscale. Copy hex values.
3. **Secondary Color** (90 seconds): Use color complementary tool, pick 60-degree opposite on color wheel or analogous.
4. **Font Pairing** (2 min): Visit fontpair.co, find one sans-serif + one body font. Example: Inter (headers) + Open Sans (body).
5. **Logo Placeholder** (2 min): Use Brandmark or LogoIpsum to generate temp logo. Replace later.
6. **Build CSS Variables** (1 min): Paste colors into `:root` CSS vars (copy from step 1-3).
7. **Apply to Figma** (1 min): Create 3 frames: color palette, typography scale, component showcase.

**Starter Code:**

```html
<style>
:root {
  --primary: hsl(215, 100%, 50%);
  --secondary: hsl(45, 100%, 50%);
  --neutral-900: #1a1a1a;
  --neutral-50: #fafafa;
  --font-body: "Inter", -apple-system, BlinkMacSystemFont, sans-serif;
  --font-heading: "Inter", sans-serif;
}

body {
  font-family: var(--font-body);
  color: var(--neutral-900);
  background: var(--neutral-50);
}

h1, h2, h3 { font-family: var(--font-heading); font-weight: 700; }
button { background: var(--primary); color: white; padding: 12px 24px; border-radius: 8px; }
</style>
```

This gives a complete, professional starting point. Refine over next sprint.

---

## Production Checklist

- [ ] Color palette tested for WCAG AA contrast (4.5:1 text)
- [ ] Typography scale built with fluid sizing (clamp())
- [ ] Spacing scale defined in CSS variables
- [ ] Dark mode tokens created and tested
- [ ] Component library established (shadcn + custom components)
- [ ] Logo variants created (full, icon, wordmark, monochrome)
- [ ] Brand guidelines document published
- [ ] Design tokens version controlled
- [ ] Storybook instance live with all components
- [ ] Icon library (Lucide) integrated
- [ ] Favicon generated and added
- [ ] OG image template created

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
