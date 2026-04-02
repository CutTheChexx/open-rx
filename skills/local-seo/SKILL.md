---
name: local-seo
description: >
  Local SEO intelligence for contractor and service-area businesses. JSON-LD
  structured data (LocalBusiness, Service, FAQ, Review schemas), service-area
  page architecture, Google Business Profile optimization, technical SEO
  checklist, meta tag patterns, and city-specific landing page strategy.
  Triggers on "SEO", "structured data", "schema markup", "JSON-LD",
  "local search", "Google Business", "meta tags", "service area pages",
  "city pages", or any local SEO task.
metadata:
  version: "1.0.0"
  sources: "schema.org, Google Search Central, local SEO guides 2026"
  author: "CutTheChexx"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Local SEO for Contractor and Service-Area Businesses

Use this playbook when building websites for service-based contractors, restoration companies, roofers, plumbers, electricians, or any business serving a geographic area. These patterns drive organic search visibility for high-intent local queries.

---

## 1. JSON-LD Structured Data Architecture

Structured data tells search engines exactly what your business is, where you operate, what you offer, and what people think of you. Use `<script type="application/ld+json">` tags in the `<head>` or before `</body>`.

### 1.1 LocalBusiness Schema (Primary Identity)

Place this on your homepage. Choose the appropriate Type: `Contractor`, `LocalBusiness`, `HomeRepair`, or `GeneralContractor`.

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "@id": "https://example.com/#business",
  "name": "Apex Home Services",
  "image": "https://example.com/logo.png",
  "description": "24/7 emergency water damage restoration, mold remediation, and reconstruction services throughout Metro Denver, Colorado.",
  "telephone": "+1-555-123-4567",
  "email": "hello@apexhome.com",
  "url": "https://example.com",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Main Street, Suite 200",
    "addressLocality": "Denver",
    "addressRegion": "CO",
    "postalCode": "80202",
    "addressCountry": "US"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 39.7392,
    "longitude": -104.9903
  },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
      "opens": "08:00",
      "closes": "17:00"
    },
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Saturday", "Sunday"],
      "opens": "09:00",
      "closes": "15:00"
    },
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"],
      "opens": "00:00",
      "closes": "23:59",
      "description": "24-hour emergency response"
    }
  ],
  "areaServed": [
    {
      "@type": "City",
      "name": "Denver",
      "addressRegion": "CO"
    },
    {
      "@type": "City",
      "name": "Aurora",
      "addressRegion": "CO"
    },
    {
      "@type": "City",
      "name": "Lakewood",
      "addressRegion": "CO"
    },
    {
      "@type": "City",
      "name": "Boulder",
      "addressRegion": "CO"
    }
  ],
  "priceRange": "$$$",
  "sameAs": [
    "https://www.facebook.com/apexhome",
    "https://www.instagram.com/apexhome",
    "https://www.yelp.com/biz/apex-home-services-denver"
  ],
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.9",
    "reviewCount": "247",
    "bestRating": "5",
    "worstRating": "1"
  },
  "hasMap": "https://maps.google.com/?q=123+Main+Street+Suite+200+Denver+CO"
}
```

### 1.2 Service Schema (Individual Services)

Create one Service schema per service offering. Place on service pages and/or the homepage.

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "@id": "https://example.com/water-damage-restoration#service",
  "name": "Water Damage Restoration",
  "description": "Professional water damage restoration, emergency mitigation, structural drying, mold assessment and remediation.",
  "serviceType": "Water Damage Restoration",
  "areaServed": [
    {
      "@type": "City",
      "name": "Denver",
      "addressRegion": "CO"
    },
    {
      "@type": "City",
      "name": "Aurora",
      "addressRegion": "CO"
    },
    {
      "@type": "City",
      "name": "Lakewood",
      "addressRegion": "CO"
    }
  ],
  "provider": {
    "@type": "LocalBusiness",
    "@id": "https://example.com/#business"
  },
  "image": "https://example.com/images/water-damage-service.jpg",
  "url": "https://example.com/water-damage-restoration",
  "priceRange": "$$$",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.9",
    "reviewCount": "52"
  }
}
```

Repeat for: Mold Remediation, Fire Restoration, Emergency Reconstruction, Structural Repair, etc.

### 1.3 FAQ Schema (for FAQ Pages)

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How quickly can you respond to a water damage emergency?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Our emergency response team is available 24/7/365. We typically arrive within 1-2 hours of your call in the Denver metro area to begin mitigation and assessment."
      }
    },
    {
      "@type": "Question",
      "name": "Does insurance cover water damage restoration?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Most homeowner's insurance policies cover sudden, accidental water damage. We work directly with your insurance adjuster and handle all documentation. However, coverage depends on your specific policy—we recommend contacting your insurer."
      }
    },
    {
      "@type": "Question",
      "name": "Can you remove mold after water damage?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes. After water mitigation, we perform mold assessment and remediation. All affected materials are treated or removed according to industry standards (IICRC guidelines)."
      }
    }
  ]
}
```

### 1.4 Review Schema (Aggregated Reviews)

```json
{
  "@context": "https://schema.org",
  "@type": "Review",
  "@id": "https://example.com/reviews/review-1#review",
  "itemReviewed": {
    "@type": "LocalBusiness",
    "@id": "https://example.com/#business"
  },
  "reviewRating": {
    "@type": "Rating",
    "ratingValue": "5",
    "bestRating": "5",
    "worstRating": "1"
  },
  "author": {
    "@type": "Person",
    "name": "Maria Johnson"
  },
  "reviewBody": "After our basement flooded, Apex Home showed up in under 90 minutes. The team was professional, efficient, and kept us informed every step of the way. Our home was restored beautifully.",
  "datePublished": "2026-03-15"
}
```

### 1.5 BreadcrumbList Schema (for Navigation)

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://example.com/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Services",
      "item": "https://example.com/services"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Water Damage Restoration",
      "item": "https://example.com/water-damage-restoration"
    },
    {
      "@type": "ListItem",
      "position": 4,
      "name": "Water Damage Restoration in Denver",
      "item": "https://example.com/water-damage-restoration-denver-co"
    }
  ]
}
```

### 1.6 Combined Schema Graph (@graph)

For pages with multiple entities, combine into a single script tag:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "LocalBusiness",
      "@id": "https://example.com/#business",
      "name": "Apex Home Services",
      "telephone": "+1-555-123-4567",
      "address": {
        "@type": "PostalAddress",
        "streetAddress": "123 Main Street, Suite 200",
        "addressLocality": "Denver",
        "addressRegion": "CO",
        "postalCode": "80202"
      }
    },
    {
      "@type": "Service",
      "@id": "https://example.com/water-damage-restoration#service",
      "name": "Water Damage Restoration",
      "provider": {
        "@id": "https://example.com/#business"
      }
    },
    {
      "@type": "BreadcrumbList",
      "itemListElement": [
        {
          "@type": "ListItem",
          "position": 1,
          "name": "Home",
          "item": "https://example.com/"
        }
      ]
    }
  ]
}
```


---

## 2. Service Area Page Architecture

The structure that ranks: service pages + city pages + service+city combo pages.

### 2.1 URL Structure

| Page Type | URL Format | Purpose |
|-----------|-----------|---------|
| Homepage | `/` | Brand identity, intro to all services |
| Service Hub | `/services` | List all services with CTAs |
| Individual Service | `/water-damage-restoration` | Deep dive: process, pricing, benefits, FAQ |
| City Landing | `/denver-co` | Overview of service area + all services available |
| Service + City | `/water-damage-restoration-denver-co` | Hyperlocal: this service in this city (highest intent) |
| Blog Post | `/blog/how-to-respond-to-water-damage` | Educational content, news, tips |
| FAQ | `/faq` | FAQPage schema, featured snippet targets |
| Reviews | `/reviews` | Testimonials, case studies, aggregated ratings |

**Rules:**
- Use lowercase, hyphens, no underscores
- No dates in URLs (avoid `/blog/2026/03/water-damage`)
- Keep structure flat when possible (avoid nesting 4+ levels deep)
- Use keyword-rich slugs, not IDs

### 2.2 Service Page Template

**File:** `/water-damage-restoration/index.html`

**Meta Tags:**
```html
<title>Water Damage Restoration in Denver, CO | Apex Home | Available 24/7</title>
<meta name="description" content="Emergency water damage restoration services available 24/7. IICRC-certified technicians, insurance-approved. Free inspection and damage assessment. Call now.">
<meta name="keywords" content="water damage restoration Denver, water damage repair CO, emergency water removal, flood damage">
<link rel="canonical" href="https://example.com/water-damage-restoration">
<meta property="og:title" content="Water Damage Restoration in Denver, CO">
<meta property="og:description" content="24/7 emergency water damage restoration. IICRC-certified, insurance-approved.">
<meta property="og:image" content="https://example.com/images/water-damage-og.jpg">
```

**H1 Formula:**
```
Emergency Water Damage Restoration in [Service Area]
```
Example: "Emergency Water Damage Restoration in Denver, Aurora & Metro Denver, CO"

**Content Structure (800+ words):**
1. **Hero Section** — Problem + solution + CTA
2. **What is Water Damage?** — Definition, types (burst pipes, leaks, floods, storm)
3. **Our Process** — Step-by-step: assessment → mitigation → drying → restoration → handoff
4. **Why Choose Us** — Certifications (IICRC), insurance relationships, 24/7 response, equipment
5. **Service Areas** — List cities, counties served with links to city+service pages
6. **FAQ Section** — 4-5 common questions (use FAQ schema)
7. **Case Studies/Before & After** — 2-3 project examples with photos
8. **Pricing/Cost Factors** — General ranges, what affects cost
9. **Emergency CTA** — "Call 555-123-4567 or submit form for 24/7 response"

**Internal Links Strategy:**
- Link to `/water-damage-restoration-denver-co`, `/water-damage-restoration-aurora-co`, etc.
- Link to `/mold-remediation` (related service)
- Link to `/blog/how-to-respond-to-water-damage` (educational)
- Link to FAQ page for each Q&A

### 2.3 City Landing Page Template

**File:** `/denver-co/index.html`

**Meta Tags:**
```html
<title>Water Damage & Restoration Services in Denver, CO | Apex Home</title>
<meta name="description" content="Serving Denver with water damage restoration, mold remediation, fire restoration, and emergency reconstruction. Available 24/7. IICRC-certified. Call today.">
<link rel="canonical" href="https://example.com/denver-co">
```

**H1:** `[Service Area] Emergency Restoration Services`
Example: "Denver Emergency Water Damage & Restoration Services"

**Content Structure (600+ words):**
1. **Intro** — "Serving Denver and [surrounding cities]"
2. **Our Services in [City]** — Grid of 4-6 services with brief descriptions and links
   - Water Damage Restoration
   - Mold Remediation
   - Fire Restoration
   - Emergency Reconstruction
3. **Why Our Denver Team** — Local expertise, response time, community involvement
4. **Service Coverage Map** — Embed Google Map showing service area
5. **Local Testimonials** — 2-3 reviews from Denver-area customers
6. **Local Resources** — Tips for Denver homeowners (winter storms, basement and crawlspace water risks, etc.)
7. **CTA** — "Call our Denver office" or form

**Internal Links:**
- `/water-damage-restoration-denver-co`
- `/mold-remediation-denver-co`
- `/fire-restoration-denver-co`
- `/denver-co/about-us` (optional local about page)

### 2.4 Service + City Combo Page

**File:** `/water-damage-restoration-denver-co/index.html` (Highest Intent = Best Conversion)

**Meta Tags:**
```html
<title>Water Damage Restoration in Denver, CO | 24/7 Emergency Response</title>
<meta name="description" content="Water damage in Denver? We arrive within 1-2 hours. IICRC-certified, insurance-approved restoration. Free assessment. Available 24/7. Call 555-123-4567.">
<link rel="canonical" href="https://example.com/water-damage-restoration-denver-co">
```

**H1:** `Water Damage Restoration in [City], [State]`
Example: "Water Damage Restoration in Denver, CO"

**Content Structure (1,000+ words):**
1. **Local Hero** — "Water damage in Denver? We respond in under 2 hours. Call 555-123-4567."
2. **Why Denver Water Damage is Urgent** — Sudden weather changes, winter storms, older basements, pipe bursts
3. **Our Denver Response Process** — Same as service page, but mention local infrastructure knowledge
4. **Common Denver Water Damage Scenarios** — Basement flooding, roof leaks (winter weather), pipe bursts, greywater, storm damage
5. **Denver Service Coverage** — Which neighborhoods, response time, local address
6. **Pricing for Denver** — General cost ranges in local context
7. **Denver Customer Reviews** — 3-5 testimonials from Denver addresses/names
8. **Emergency CTA** — Prominent phone number and form

**Internal Links:**
- `/water-damage-restoration` (parent service page)
- `/denver-co` (parent city page)
- `/mold-remediation-denver-co` (related service in same city)
- `/blog/water-damage-in-denver-winter-storm-season` (local blog post)

---

## 3. Meta Tag Patterns

Consistent, keyword-optimized meta tags on every page drive CTR from search results.

### 3.1 Title Tag Formula

**Pattern:**
```
[Service] in [City], [State] | [Brand] | [Trust Signal]
```

**Examples:**
- Water Damage Restoration in Denver, CO | Apex Home | 24/7 Emergency
- Mold Remediation in Aurora, CO | Apex Home | IICRC Certified
- Fire Restoration in Boulder, CO | Apex Home | 4.9★ Rating
- Best Roofer in Lakewood, CO | Apex Roofing | Licensed & Insured

**Rules:**
- Keep under 60 characters for desktop, 50 for mobile
- Put city/state early (geo-targeting)
- Include trust signal (24/7, certified, ratings, insured)
- Use pipe (|) or dash (-) as separator

### 3.2 Meta Description Formula

**Pattern:**
```
[Service offering + benefit]. [Process or guarantee]. [CTA].
```

**Example:**
```html
<meta name="description" content="Water damage restoration with 24/7 emergency response. IICRC-certified team, insurance-approved. Free inspection. Call 555-123-4567 or book online.">
```

**Rules:**
- 150-160 characters (displays in full on mobile)
- One action verb ("restore," "repair," "solve")
- Include phone number OR form CTA (but not both—pick strongest)
- Include location (city, state)
- Include key benefit (24/7, certified, licensed, insured)

### 3.3 Open Graph Tags (Social Sharing)

```html
<meta property="og:title" content="Water Damage Restoration in Denver, CO | 24/7">
<meta property="og:description" content="Emergency water damage restoration. Available 24/7. IICRC-certified. Free assessment.">
<meta property="og:image" content="https://example.com/images/og-water-damage.jpg">
<meta property="og:url" content="https://example.com/water-damage-restoration-denver-co">
<meta property="og:type" content="website">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Water Damage Restoration in Denver, CO">
<meta name="twitter:description" content="Emergency water damage restoration. Available 24/7.">
<meta name="twitter:image" content="https://example.com/images/og-water-damage.jpg">
```

**Image Spec:** 1200x630px, branded, shows service (e.g., restoration team, before/after)

### 3.4 Canonical Tags

```html
<!-- Service page -->
<link rel="canonical" href="https://example.com/water-damage-restoration">

<!-- City+service combo page -->
<link rel="canonical" href="https://example.com/water-damage-restoration-denver-co">

<!-- Category/hub page (self-referential) -->
<link rel="canonical" href="https://example.com/services">
```

Use canonical on all pages, even to themselves, to prevent duplicate content issues.

### 3.5 Robots Meta

```html
<!-- Default (index & follow) -->
<meta name="robots" content="index, follow">

<!-- Noindex for duplicate/thin pages -->
<meta name="robots" content="noindex, follow">

<!-- Prevent search console sidebar ads -->
<meta name="robots" content="noindex, nofollow">
```

---

## 4. Technical SEO Checklist

Every contractor site must pass these checks. Use Lighthouse (Chrome DevTools), Google Search Console, and PageSpeed Insights to validate.

### 4.1 Mobile & Responsive Design

- [ ] Viewport meta tag: `<meta name="viewport" content="width=device-width, initial-scale=1">`
- [ ] All content readable on mobile (no horizontal scroll)
- [ ] Buttons/CTAs are 48x48px minimum (touch targets)
- [ ] Navigation is hamburger or sticky on mobile
- [ ] Forms have large inputs and proper spacing
- [ ] Test on real devices: iPhone 12/13, Samsung Galaxy S21+

### 4.2 Core Web Vitals (Google's ranking signals)

| Metric | Threshold | Action |
|--------|-----------|--------|
| LCP (Largest Contentful Paint) | < 2.5s | Optimize hero image, preload fonts |
| INP (Interaction to Next Paint) | < 200ms | Defer non-critical JS, optimize handlers |
| CLS (Cumulative Layout Shift) | < 0.1 | Use aspect-ratio on images/video, avoid surprise ads |

**Quick wins:**
- Use `<img loading="lazy">` for below-fold images
- Serve images in WebP format (with JPG fallback)
- Preload hero image: `<link rel="preload" as="image" href="/hero.webp">`
- Minify CSS/JS
- Use async/defer on scripts

### 4.3 HTTPS & Security

- [ ] SSL certificate installed (free via Let's Encrypt)
- [ ] All images/resources load over HTTPS (no mixed content warnings)
- [ ] Security.txt file: `/.well-known/security.txt`
- [ ] X-Content-Type-Options: `nosniff` header

### 4.4 XML Sitemap

Create `/sitemap.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2026-03-31</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/water-damage-restoration</loc>
    <lastmod>2026-03-31</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.9</priority>
  </url>
  <url>
    <loc>https://example.com/water-damage-restoration-denver-co</loc>
    <lastmod>2026-03-31</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.9</priority>
  </url>
</urlset>
```

Submit to Google Search Console.

### 4.5 robots.txt

Create `/robots.txt`:

```
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /thank-you/
Disallow: /cart/

Sitemap: https://example.com/sitemap.xml
```

### 4.6 Image Alt Text

Every image needs descriptive alt text:

```html
<!-- Bad -->
<img src="photo.jpg" alt="">

<!-- Good -->
<img src="water-damage-bathroom.jpg" alt="Water-damaged bathroom with drywall removal in progress">

<!-- Service photo alt template -->
<img src="water-extraction-equipment.jpg" alt="Professional water extraction equipment responding to water damage in Denver home">
```

### 4.7 Heading Hierarchy

- [ ] Exactly ONE H1 per page
- [ ] H1 is page title (e.g., "Water Damage Restoration in Denver, CO")
- [ ] H2s are major sections (e.g., "Our Emergency Response Process")
- [ ] H3s nest under H2s (no skipping levels)
- [ ] Never use heading tags for styling (use CSS)

```html
<h1>Water Damage Restoration in Denver, CO</h1>
<h2>What is Water Damage?</h2>
<h3>Types of Water Damage</h3>
<h2>Our Process</h2>
<h3>Step 1: Assessment</h3>
<h3>Step 2: Mitigation</h3>
```

### 4.8 Internal Linking

- [ ] Every page reachable within 3 clicks from homepage
- [ ] Service pages link to city pages
- [ ] City pages link to service pages
- [ ] Blog posts link to service pages (contextual)
- [ ] Use descriptive anchor text: "Water damage restoration in Denver" not "click here"
- [ ] No orphaned pages (pages with zero internal links to them)

### 4.9 404 Error Page

Create `/404.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Page Not Found | Apex Home</title>
  <meta name="robots" content="noindex">
</head>
<body>
  <h1>Page Not Found</h1>
  <p>The page you're looking for doesn't exist. Here's where you can go:</p>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/services">Our Services</a></li>
    <li><a href="/water-damage-restoration">Water Damage Restoration</a></li>
    <li><a href="/contact">Contact Us</a></li>
  </ul>
  <p>Or call us at <a href="tel:+15551234567">555-123-4567</a></p>
</body>
</html>
```

### 4.10 Page Speed Optimizations

| Technique | Benefit |
|-----------|---------|
| Image lazy loading (`loading="lazy"`) | Reduce initial page weight |
| WebP format + JPG fallback | 25-35% smaller images |
| CSS minification & critical CSS inlining | Faster render |
| JavaScript deferral (defer/async) | Faster FCP |
| Preconnect to external domains | Reduce DNS latency |
| Gzip compression (server-level) | Reduce transfer size |

---

## 5. Google Business Profile Optimization

Free local search visibility. Mandatory for contractors.

### 5.1 Profile Completeness Checklist

- [ ] Business name (match website H1)
- [ ] Phone number (verified SMS/call)
- [ ] Service area (check box for each city served, or draw custom polygon)
- [ ] Hours of operation (including emergency/after-hours if 24/7)
- [ ] Website (HTTPS, home page URL)
- [ ] Category (Primary: "Water Damage Restoration" | Secondary: "Restoration Service", "Emergency Services")
- [ ] Address (physical location; use service area if no storefront)
- [ ] Website and service area filled completely (not just address)

### 5.2 Attributes & Descriptions

Add service-specific attributes in GBP:

```
Services Offered:
✓ 24-hour service
✓ Water damage restoration
✓ Mold remediation
✓ Emergency reconstruction
✓ Insurance claims assistance
✓ IICRC certified
✓ Eco-friendly processes
```

Write a compelling business description (750 chars max):

> Apex Home Services is Metro Denver's trusted emergency response partner. Available 24/7/365 with response times under 2 hours. Our IICRC-certified team specializes in water damage restoration, mold remediation, fire restoration, and structural reconstruction. We work directly with insurance companies and manage all documentation. Serving Denver, Aurora, Lakewood, Boulder, and surrounding areas.

### 5.3 Photos Strategy

Minimum 50 photos. Organize by category:

- **Hero/Branding** (5): Professional team photos, office, equipment trucks, logos
- **Project Before/After** (20): Water damage, mold remediation, fire damage—before and after side-by-sides
- **Equipment & Process** (10): Water extraction, structural drying, dehumidifiers, assessment tools
- **Team & Credentials** (5): Certified technicians, team photos, credentials on display
- **Service Areas** (10): Photos from different cities served (Denver landmark, Aurora street, etc.)

**Photo naming:** `water-damage-basement-before.jpg`, not `IMG_001.jpg`

### 5.4 Reviews Management

- [ ] Respond to EVERY review within 24 hours (positive and negative)
- [ ] Use consistent tone (professional, empathetic, solution-focused)
- [ ] Ask happy customers for reviews (email after project close: "We'd love a 5-star review!")
- [ ] Never ask for specific star ratings (against policy)
- [ ] On negative reviews: apologize, take offline, resolve, ask for update
- [ ] Q&A section: Pre-populate 10-15 common questions and answers

**Pre-populated Q&A Examples:**

> Q: How quickly can you respond?
> A: Our 24/7 emergency team arrives within 1-2 hours of your call.

> Q: Do you work with insurance?
> A: Yes, we're approved by all major carriers and handle all documentation.

> Q: Are you IICRC certified?
> A: Yes, all technicians hold current IICRC certifications.

### 5.5 Posts Strategy

Post weekly (before taking a break in slow season). Use GBP's native Posts feature:

**Post Types:**

| Type | Frequency | Example |
|------|-----------|---------|
| Before/After Projects | 2x/week | Photo carousel of completed water damage job |
| Tips & Education | 1x/week | "Winter Pipe Burst Prevention: 5 Tips" |
| Seasonal Content | Weekly | "Winter Storm Water Damage Tips" (Nov-Mar in CO) |
| Team Highlights | 2x/month | Technician spotlight, training, certifications |
| Q&A from reviews | 1x/month | Answer a frequently asked question |
| Promotions | 1-2x/month | "Free inspection for new customers" |

---

## 6. Content Strategy for Organic Rankings

Build pages targeting different keywords and intent. Publish consistently.

### 6.1 Content Roadmap

| Page Type | Target Keywords | Word Count | Update Frequency |
|-----------|-----------------|-----------|------------------|
| Service pages | "[Service] + location variations" | 800-1,500 | Quarterly |
| City landing pages | "[City] + service umbrella" | 600-1,000 | Quarterly |
| City+service combo | "[Service] in [City]" (highest intent) | 1,000-1,500 | Quarterly |
| FAQ page | "How to...", "Cost of..." | 1,500+ with schema | Quarterly |
| Blog posts | Long-tail ("water damage from burst pipes") | 1,500-3,000 | Weekly |
| About page | Brand, team, credentials | 600-800 | Annual |
| Reviews/testimonials | Social proof | 500+ | Ongoing |

### 6.2 Blog Post Targets

Create 1-2 posts per week targeting "how-to" and "cost" queries:

**Examples:**

- "How to Respond to Water Damage in Your Home (First 24 Hours)"
- "Cost of Water Damage Restoration in Denver: 2026 Pricing Guide"
- "Mold After Water Damage: When to Call a Professional"
- "Can I Stay in My Home During Water Damage Restoration?"
- "Winter Pipe Bursts: Prevention and Emergency Response"
- "Water Damage from Ceiling Leaks: What to Do Immediately"

Each blog post should:
- Link to related service pages (2-3 contextual links)
- Answer the query completely (steal Google features)
- Include 1-2 images with alt text
- Use H2/H3 hierarchy
- Include CTA at end ("Need emergency water damage restoration? Call 555-123-4567")
- Target local keywords when relevant

### 6.3 FAQ Page Structure

Use FAQPage schema. Targets featured snippet positions. Include 15-20 questions:

**Question Categories:**
- **Emergency Response** (How quickly...? What if it's after hours...?)
- **Coverage & Costs** (Does insurance...? What affects pricing...?)
- **Process** (What happens first...? How long does restoration...?)
- **Mold** (Can you find mold...? Is mold dangerous...?)
- **Prevention** (How can I prevent...? What's the best...?)
- **Credentials** (Are you certified...? Are you licensed...?)

Each answer: 50-150 words, direct and actionable.

### 6.4 About Page

Build trust with team, credentials, community involvement:

**Sections:**
1. **Who We Are** — Year founded, mission statement
2. **Leadership Team** — Names, photos, bios (include certifications)
3. **Certifications & Training** — IICRC, ANSI, state licenses, ongoing education
4. **Community Involvement** — Donations, sponsorships, volunteer work
5. **Timeline/Awards** — 10-year milestones, local business awards
6. **Team Photos** — Genuine candids of the team working (not stock photos)

Include schema: `@type: Organization` with founder info, founding date, award fields.

---

## 7. Ongoing Maintenance & Monitoring

SEO is not one-time; check monthly:

### Monitoring Checklist

- [ ] Google Search Console: Check impressions, clicks, average position for service pages
- [ ] Core Web Vitals: Use PageSpeed Insights—target all green (LCP < 2.5s, INP < 200ms, CLS < 0.1)
- [ ] Google Business Profile: Verify hours, photos, reviews (respond to all)
- [ ] Ranking keywords: Service pages should rank #1-5 for "[Service] in [City]"
- [ ] Traffic trends: Seasonal spikes during winter months, seasonal storms
- [ ] Backlinks: Monitor for lost links (Google Search Console)
- [ ] New content: Blog posts, photos in GBP, fresh testimonials

### Quick Wins (First Month)

1. Set up Google Business Profile completely (photo upload, service area, hours)
2. Add LocalBusiness + Service JSON-LD to homepage
3. Build 4-5 city+service combo pages for top markets
4. Publish FAQ page with FAQPage schema
5. Create 4-8 blog posts targeting "how-to" and "cost" queries
6. Submit XML sitemap to Google Search Console
7. Fix Core Web Vitals (image optimization, JS deferral)

---

## Usage Notes

This playbook applies to contractors, restoration companies, HVAC, plumbing, electrical, handyman services, and any business with a defined service area. Adapt city names, services, and keywords to your market. Prioritize high-intent keywords ("[Service] in [City]") over brand keywords. Publish consistently (at least 2 blog posts per month) to compound SEO gains. Local SEO results take 3-6 months; stay patient and measure monthly via Search Console.

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
