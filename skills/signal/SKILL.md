---
name: Signal
description: Universal SEO and online visibility compound skill — Google Business Profile optimization, local search, structured data, review strategy, and technical SEO for any business.
version: 1.0.0
author: CutTheChexx
category: SEO
tags:
  - local-seo
  - google-business-profile
  - structured-data
  - review-strategy
  - technical-seo
  - meta-tags
  - content-seo
  - service-area-pages
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Signal: The SEO Prescription for Every Business

Signal is your treatment plan for visibility. Whether you run a dental office, construction company, restaurant, or SaaS platform, being invisible online means you don't exist. Signal covers the architecture, mechanics, and strategy that make businesses searchable, trustworthy, and found.

## Core Problem

Your business is being buried because:
- You don't control your online identity
- Search engines can't understand what you do
- You're bleeding leads to competitors with better visibility
- Your review reputation is unmanaged or negative
- Your website is broken to search engines, even if it looks fine

Signal fixes this.

---

## Part 1: Google Business Profile — Your Foundation

### What This Is

Google Business Profile (GBP) is the single most important property you control for local search. It's the knowledge panel that shows on Google Maps and in search results. 95% of local searches lead here first.

### Setup Checklist

1. **Create or claim your profile**
   - Go to `google.com/business`
   - Search for your business
   - If it exists (likely), claim it
   - If not, create it
   - Verify ownership (postcard, phone, or email depending on business type)

2. **Complete every field**
   - Business name (match your legal documents and signage exactly — NAP consistency)
   - Business category (primary + up to 9 secondaries)
   - Address (must be a real, serviceable location if you're local)
   - Phone number (dedicated business line if possible, not shared)
   - Website URL
   - Hours of operation (include all holidays)
   - Service area (if you don't have a physical location)

3. **Add photos and videos**
   - 10+ interior/exterior photos minimum
   - Rotate them monthly
   - Show the space, team, products, process
   - 2-3 short videos (15-30 seconds) of your operation

4. **Write the description**
   - 750 characters maximum
   - Answer: What do you do, who do you serve, why are you different?
   - Example (plumber): "Licensed plumber serving downtown and suburbs for 12 years. We specialize in emergency repair, drain cleaning, and water heater installation. Same-day service available. Upfront pricing, no hidden fees."

### NAP Consistency

NAP = Name, Address, Phone. It must be identical everywhere.

Where it appears:
- Your website footer
- Google Business Profile
- Google Maps
- Yelp, Facebook, LinkedIn
- Industry directories (HVAC contractor directories, dentist registries, etc.)
- Local citations (Yellowpages, Angie's List, etc.)

Mismatch = search ranking penalty. Audit it:

```
Business Name    |  Website  |  GBP   |  Yelp  |  Facebook
Mike's Plumbing  |  mike's   |  Mike's|  Mike |  Mikes ❌
                 |  plumbing | Plumbing| Plumbing
```

Fix it now.

---

## Part 2: Local Citations and Authority

A citation is any mention of your business name + address + phone on another website, especially directories.

### High-Value Citations (Build These First)

1. **Google Business Profile** (already doing this)
2. **Yelp** — Create a business account, claim your listing
3. **Apple Maps** — Apply for your business
4. **Facebook Business Page** — Complete every detail
5. **Industry-specific directories:**
   - Dentist? Zocdoc, Healthgrades, Doctoralia
   - Lawyer? Avvo, Justia, Findlaw
   - HVAC? Angie's List, ServiceMaster, The Blue Book
   - Restaurant? Michelin, Eater, local food critic sites
   - Local services? Home Advisor, Thumbtack, Care.com

### How Citations Work

Each citation is a vote. More votes = higher rankings. But they only count if:
- NAP is consistent
- Website is linked back
- Source is relevant to your industry
- Profile is complete (not a stub)

### Quick Citation Audit

1. Google: `"Your Business Name"`
2. Check first 50 results
3. List all directories/citations
4. Verify NAP matches across all
5. Add your website link where possible

---

## Part 3: Structured Data / JSON-LD Schema

This is how you tell Google exactly what you are in machine-readable format. Without it, Google guesses.

### Why It Matters

Structured data enables:
- Rich snippets in search results (star ratings, prices, availability)
- Knowledge panel details
- Voice search understanding
- Schema validation in Google Search Console

### Essential Schemas for Every Business

#### LocalBusiness (Foundation)

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Parkside Dental",
  "image": "https://example.com/logo.png",
  "description": "Full-service dental practice offering cleanings, restorations, and cosmetic work.",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Main Street",
    "addressLocality": "Denver",
    "addressRegion": "CO",
    "postalCode": "80202",
    "addressCountry": "US"
  },
  "telephone": "+1-303-555-0100",
  "url": "https://example.com",
  "sameAs": [
    "https://www.facebook.com/parksidedental",
    "https://www.yelp.com/biz/parkside-dental"
  ],
  "priceRange": "$$",
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": "Monday",
      "opens": "09:00",
      "closes": "17:00"
    },
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": "Saturday",
      "opens": "10:00",
      "closes": "14:00"
    }
  ]
}
```

#### Service (If You Offer Multiple Services)

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "Root Canal Therapy",
  "description": "Complex root canal treatment for infected teeth.",
  "provider": {
    "@type": "LocalBusiness",
    "name": "Parkside Dental"
  },
  "areaServed": {
    "@type": "City",
    "name": "Denver"
  },
  "availableLanguage": "en",
  "potentialAction": {
    "@type": "ReserveAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://example.com/book"
    }
  }
}
```

#### AggregateRating + Review Schema

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Parkside Dental",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "reviewCount": "127",
    "bestRating": "5",
    "worstRating": "1"
  },
  "review": [
    {
      "@type": "Review",
      "author": {
        "@type": "Person",
        "name": "Sarah M."
      },
      "datePublished": "2025-10-15",
      "reviewRating": {
        "@type": "Rating",
        "ratingValue": "5"
      },
      "reviewBody": "Dr. Chen was incredibly thorough and explained everything. No surprises on the bill. Highly recommend."
    }
  ]
}
```

#### FAQ Schema

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How often should I get my teeth cleaned?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Most people should have a professional cleaning every 6 months. If you have gum disease, your dentist may recommend every 3 months."
      }
    },
    {
      "@type": "Question",
      "name": "Do you offer dental insurance plans?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "We accept all major insurance plans. Check our insurance page for a complete list, or call us to verify your coverage."
      }
    }
  ]
}
```

### Implementation

1. **Use Google's Structured Data Testing Tool** (`schema.org/tools/json-ld-generator`)
2. **Add to your site**
   - In `<head>` tag as `<script type="application/ld+json">`
   - Or use plugin (Yoast SEO, Schema Pro, etc.)
3. **Validate in Google Search Console** (Rich Results tab)
4. **Monitor for errors** (monthly)

---

## Part 4: Service Area Pages (Multi-Location Strategy)

If you serve multiple cities or neighborhoods, service area pages are critical.

### Architecture

For a plumber serving Denver suburbs:

```
/service-areas/ (hub page, lists all cities)
  ├─ /service-areas/denver/ (city page with local content)
  ├─ /service-areas/aurora/
  ├─ /service-areas/littleton/
  └─ /service-areas/colorado-springs/
```

### Service Area Page Template

**URL:** `domain.com/service-areas/denver/`

**Title Tag (60 chars):** `Plumbing Services in Denver, CO | 24/7 Emergency Repair`

**Meta Description (160 chars):** `Licensed plumber in Denver offering emergency repair, drain cleaning, and water heater installation. Same-day service. Call now for a free estimate.`

**H1:** `Plumbing Services in Denver, Colorado`

**Body Content (800-1200 words):**

1. **Localized intro paragraph** (100-150 words)
   - Mention the specific city by name
   - Acknowledge local landmarks, neighborhoods, or geography
   - Example: "For over 12 years, we've served Denver's Capitol Hill, Highlands, and Washington Park neighborhoods, fixing burst pipes, unclogging drains, and replacing water heaters the same day you call."

2. **Services offered** (with local context)
   - Emergency repair (24/7 response in Denver area)
   - Drain cleaning (common in old Denver homes)
   - Water heater replacement (local brand recommendations: Rheem, AO Smith)

3. **Local service area details** (neighborhoods, zip codes)
   - "We serve zip codes: 80202, 80203, 80210, 80211..."

4. **Local credentials**
   - Colorado state license number
   - Denver business registration
   - Specific years of service in Denver area

5. **Call-to-action** (book/call button)
   - "Schedule your Denver plumbing appointment"

6. **Local review testimonials** (if you have them)

7. **FAQ section specific to Denver**
   - "Why do Denver homes need drain cleaning more often?" (mineral content)
   - "What's the cost of emergency plumbing in Denver?"

### Avoid These Mistakes

- **Duplicate content** across service pages — Each needs 70%+ unique content
- **Auto-generated city lists** — Google penalizes thousands of thin pages for the same boilerplate
- **No actual service location** — If you don't actually serve that area, don't create a page
- **Keyword stuffing** — "Denver plumber Denver Colorado plumbing Denver" is spam

### Schema for Service Area Pages

Add to each service area page:

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "ServiceName",
  "areaServed": {
    "@type": "City",
    "name": "Denver"
  },
  "geo": {
    "@type": "GeoShape",
    "box": "39.5501 -104.8202 39.7143 -104.5743"
  }
}
```

---

## Part 5: Review Strategy and Reputation Management

Reviews are ranking factors AND conversion drivers. A 4.5-star business converts 2x better than a 3.5-star business, even in the same industry.

### Getting More Reviews

#### Systematic Approach (The Right Way)

1. **Make it a business process**
   - At checkout (ecommerce): Email link to review immediately
   - At appointment end (services): Verbal + SMS request
   - In invoice/receipt: QR code to review page
   - 48 hours later: Email reminder if no review yet

2. **Where to ask for reviews**

   **For local businesses:**
   - Google (highest priority)
   - Yelp
   - Facebook
   - Industry-specific (Healthgrades for dentists, Avvo for lawyers, etc.)

   **For ecommerce:**
   - Google Customer Reviews
   - Amazon reviews (if you sell there)
   - Trustpilot
   - Your own site (Yotpo, Reviews.io, etc.)

3. **The request template**

   Email:
   ```
   Subject: How was your experience at [Business Name]?

   Hi [Name],

   Thanks for choosing us on [date]. We'd love to hear about your experience.

   Your feedback helps other customers decide, and helps us improve.

   ⭐ Leave a review on Google
   [short.url/review-google]

   Takes 60 seconds. That's it.

   Thanks,
   [Owner Name]
   ```

4. **Timing matters**
   - Ask immediately after positive experience (while emotions are high)
   - Wait 48 hours for services (let them actually use the service)
   - Never ask after complaints
   - Don't ask too frequently (max once per customer per 6 months)

#### What NOT To Do (Black Hat Review Tactics)

- **Fake reviews** — Google catches these. Manual penalty = death
- **Review gating** — "Only leave reviews if 5 stars" — Violates Google policy
- **Paying for reviews** — "Leave a 5-star review, get $10 off" — Illegal in most jurisdictions
- **Replying with incentives** — "Thanks for 5 stars! Mention this review for 20% off" — Illegal
- **Deleting negative reviews** — You can't. Don't try
- **Impersonating customers** — Writing reviews as them — Criminal fraud

### Responding to Reviews

Every review deserves a response. Public conversations show you're engaged.

**Template for 5-star reviews:**

```
Thank you, [Name]! We truly appreciate you taking the time to share your experience.
Looking forward to serving you again. ⭐
```

**Template for 1-2 star reviews:**

```
Hi [Name], we're sorry to hear that [specific complaint from review].
This isn't our standard. Please contact us directly at [phone/email] —
we'd like to make this right.
```

**Key rules:**
- Always respond publicly (don't hide it)
- Never get defensive or argue
- Offer solutions, not excuses
- Include contact info for follow-up
- Respond within 24-48 hours

### Review Schema Markup

Add to your homepage and service pages:

```json
{
  "@context": "https://schema.org",
  "@type": "AggregateRating",
  "ratingValue": "4.8",
  "reviewCount": "247",
  "name": "Customer Satisfaction Rating"
}
```

This displays stars in search results.

---

## Part 6: Meta Tags — The Title and Description Battleground

Meta tags are the first impression in search results. They control click-through rate.

### Title Tags

**Best practices:**
- 50-60 characters (Google truncates at ~60)
- Lead with your unique value
- Include primary keyword
- Include location if local
- Make it click-worthy

**Examples:**

Bad:
```
Home | Website Builder
```

Good:
```
Website Builder for Small Businesses | Fast, No Code Required
```

Bad:
```
Dentist
```

Good:
```
Emergency Dental Care in Denver | Same-Day Appointments
```

**Template (local services):**
```
[Service] in [City] | [Unique Promise]
```

### Meta Descriptions

**Best practices:**
- 150-160 characters
- Answer "why should I click?"
- Include call-to-action when relevant
- Don't repeat title
- One per page (no duplicates)

**Examples:**

Bad:
```
Welcome to our website. We offer dental services.
```

Good:
```
Need emergency dental care in Denver? We see emergency patients same-day. Open weekends. Call now for pricing.
```

**Template:**
```
[Problem solved] in [location]. [Proof/benefit]. [CTA].
```

### Open Graph Tags (Social Sharing)

Add to `<head>`:

```html
<meta property="og:title" content="Emergency Dental Care in Denver" />
<meta property="og:description" content="Same-day appointments, no wait times, upfront pricing." />
<meta property="og:image" content="https://example.com/featured-image.jpg" />
<meta property="og:url" content="https://example.com/page" />
<meta property="og:type" content="website" />
```

### Twitter Card Tags

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Emergency Dental Care in Denver" />
<meta name="twitter:description" content="Same-day appointments, no wait times." />
<meta name="twitter:image" content="https://example.com/featured-image.jpg" />
```

### Canonical Tags

**Why:** Prevent duplicate content penalties

Add to every page:

```html
<link rel="canonical" href="https://example.com/page" />
```

**When it matters:**
- `example.com/page` and `example.com/page/` (trailing slash)
- `example.com/page?id=123` (URL parameters)
- `example.com` and `www.example.com` (trailing www)

Pick one version of your site and canonicalize to it.

---

## Part 7: Technical SEO Checklist

Google's crawlers need to read your site. Bad technical SEO kills visibility even with great content.

### Site Speed (Core Web Vitals)

Google ranks fast sites higher. Check yours:
- **Tool:** `PageSpeed Insights` (pagespeed.web.dev)
- **Target:** 80+ score

**Quick fixes:**
- Compress images (use .webp format)
- Enable caching (browser caching, server-side caching)
- Minify CSS/JavaScript
- Lazy-load images (especially below the fold)
- Use a CDN (Cloudflare, Fastly, CloudFront)
- Upgrade hosting if server is slow

### Mobile-First Indexing

Google primarily crawls and ranks the mobile version of your site. If your site is broken on mobile, you're penalized.

**Checklist:**
- Responsive design (site adjusts to screen size)
- No mobile pop-ups blocking content
- Touch targets at least 48x48 pixels
- Fast on mobile (test with PageSpeed on mobile)
- Readable font size (at least 16px base)

### Crawlability

Google needs to be able to crawl your site.

**robots.txt** (place at root: `domain.com/robots.txt`):

```
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /private/
Disallow: /*.pdf$

Sitemap: https://example.com/sitemap.xml
```

**What not to do:**
- Don't block Google with robots.txt (`User-agent: * Disallow: /`)
- Don't require login to access pages
- Don't require JavaScript to load content (Google struggles with this)

### Sitemap

**XML Sitemap** (submit to Google Search Console):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2025-04-05</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://example.com/services/</loc>
    <lastmod>2025-04-05</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.9</priority>
  </url>
</urlset>
```

Refresh monthly. Most plugins auto-generate this.

### HTTPS

**Non-negotiable.** Google penalizes non-HTTPS sites.

- Buy SSL certificate (Let's Encrypt is free)
- Redirect all HTTP to HTTPS
- Test at `sslchecker.com`

### Duplicate Content

**Problem:** Same content on multiple URLs = search penalty

**Common causes:**
- Session IDs in URLs (`example.com/page?sid=123`)
- Trailing slash inconsistency
- WWW and non-WWW versions both indexable
- Pagination (`?page=1`, `?page=2`, etc.)

**Solution:**
- Set preferred domain in Google Search Console
- Use canonical tags
- Redirect non-preferred versions

### Indexation

**Check what Google has indexed:**

1. Go to Google Search Console
2. Coverage > Indexed pages
3. Look for errors (not indexed)
4. "Excluded" tab — make sure intentional pages aren't excluded

**If pages not indexed:**
- Check robots.txt (not blocking?)
- Check noindex meta tag (accidental?)
- Check redirect chains
- Submit URL directly to Search Console
- Wait 2 weeks

---

## Part 8: Content SEO — The Engine That Drives Everything

Content is where strategy meets execution. Bad content = invisible. Good content = found AND converts.

### Keyword Research Framework

**Step 1: Brain dump**
- What questions do customers ask you?
- What problems do you solve?
- What are you the expert in?

Example (dentist):
- "How much does a root canal cost?"
- "Is teeth whitening safe?"
- "Do I need a crown or filling?"
- "How often should I see a dentist?"

**Step 2: Tool validation**
- Google Keyword Planner (free, but limited)
- Semrush (paid, but comprehensive)
- Ahrefs (paid)
- Answer the Public (free for limited searches)

**Step 3: Find low-competition keywords**
- High search volume + low competition = gold
- Example: "dentist in [your city]" likely has less competition than "dentist"
- Long-tail keywords (3+ words) are easier to rank for

### Content Clusters (The Modern Approach)

Instead of isolated blog posts, organize content in clusters:

**Hub Page** (pillar content)
- Topic: "Complete Guide to Dental Fillings"
- 2000-3000 words
- Covers every angle
- Links to cluster pages

**Cluster Pages** (supporting content)
- "Silver Fillings vs. Composite Fillings"
- "How Long Do Fillings Last?"
- "Cost of Dental Fillings in Denver"
- Each links back to hub and to each other

**Why it works:**
- Google sees you as an authority on the topic
- Users stay longer on your site
- Internal linking passes authority
- Rankings improve faster

### Content Template (High-Converting)

**For service businesses:**

```
H1: [Problem] Solutions for [Location]

Intro (100-150 words)
- Acknowledge the pain
- Promise the solution
- Why readers should trust you

Problem Deep-Dive (300-400 words)
- Explain the problem in detail
- Common misconceptions
- Why it matters

Solution/Service (400-600 words)
- How you solve it step-by-step
- What makes your approach different
- Client results/testimonials

Pricing/Next Steps (150-200 words)
- Transparent pricing (build trust)
- How to get started
- CTA (call, email, book appointment)

FAQ (150-300 words)
- 3-5 common questions
- Clear, direct answers

```

### Blog Strategy That Actually Works

**Don't blog just to blog.**

**Do blog to answer questions your customers are already searching for.**

**Monthly publishing plan for a local service business:**

- Week 1-2: Research keywords (20 searches/questions from customers)
- Week 2-3: Write 2-3 blog posts (1500-2500 words each)
- Week 3-4: Optimize + publish + promote (email, social, Google)

**Why 2-3/month, not daily?**
- Quality > quantity
- Consistency builds authority
- Each post needs promotion
- Time to rank (3-6 months per post typically)

### Internal Linking

**Strategy:** Link to your high-value pages from as much content as possible.

**Example:**
- Root cause of bad breath? → Links to "Preventive Cleaning" service page
- Comparing whitening methods? → Links to "Professional Whitening" service page

**Best practices:**
- Use descriptive anchor text (not "click here")
- Link when contextually relevant
- Link to pages you want to rank
- 2-3 internal links per 1000 words of content

---

## Part 9: Google Search Console — Your Diagnostic Tool

GSC is how you monitor, debug, and improve your SEO.

### Setup

1. Go to `search.google.com/search-console`
2. Add your property (domain)
3. Verify ownership (DNS record, HTML file, or Google Analytics)
4. Add your sitemap (`domain.com/sitemap.xml`)

### Monthly Monitoring

**Performance Tab:**
- Which keywords bring traffic?
- Which pages get clicked?
- What's your average ranking position?
- What's your click-through rate?

**Coverage Tab:**
- How many pages indexed?
- Any errors? (Fix immediately)
- Any warnings? (Address)

**Mobile Usability Tab:**
- Any mobile issues? (Fix immediately)
- Google prioritizes mobile

**Enhancements Tab:**
- Rich results eligible?
- Structured data errors?

### Common Issues to Fix

| Issue | Fix |
|-------|-----|
| Page indexed but gets 0 clicks | Improve title/description (CTR issue, not ranking) |
| Page not indexed | Check robots.txt, noindex tag, crawl errors |
| Mobile usability errors | Test on PageSpeed Insights, fix layout issues |
| Structured data errors | Validate JSON-LD, remove broken markup |
| Site crawl errors | Check server errors (500, 403), fix redirects |

---

## Part 10: Anti-Patterns — What NOT To Do

These will bury you.

### Black Hat Tactics (Instant Penalties)

- **Cloaking:** Show different content to Google than users — Banned
- **Private blog networks:** Buying links from spammy networks — Banned
- **Keyword stuffing:** Repeating keywords unnatural ("dentist dentist dentist") — Penalized
- **Doorway pages:** Thin pages designed only for search, not users — Banned
- **Auto-generated content:** AI without human review for SEO manipulation — Risky
- **Fake reviews:** Paying for 5-star reviews or writing fake ones — Illegal
- **Hacked sites with spam links:** Lose all ranking — Fix immediately

### Gray Hat Mistakes (Risky, Likely Penalties)

- **Excessive guest posting:** 100s of guest posts solely for backlinks — Penalized
- **Over-optimization:** Every page targets the same keyword — Penalized
- **Thin content:** Fluff pages under 300 words with no value — Penalized
- **Duplicate content across domains:** Owning multiple sites with identical content — Penalized
- **Manipulated anchor text:** All backlinks say "best dentist Denver" — Penalized
- **Private blog networks:** Networks of sites linking to yours for money — Penalized

### Common Honest Mistakes

1. **Duplicate content**
   - Problem: Same content on multiple pages
   - Fix: Use canonical tags, 301 redirects, set preferred domain in GSC

2. **Missing alt text on images**
   - Problem: Google can't read images without description
   - Fix: `<img src="photo.jpg" alt="Root canal procedure diagram" />`

3. **No mobile optimization**
   - Problem: Site doesn't work on phones
   - Fix: Use responsive design, test on mobile, use PageSpeed Insights

4. **Slow site speed**
   - Problem: Pages take 5+ seconds to load
   - Fix: Compress images, enable caching, upgrade hosting, use CDN

5. **Not responding to reviews**
   - Problem: Customers see you don't care
   - Fix: Response rate is a ranking factor; respond to all reviews

6. **Inconsistent NAP**
   - Problem: Business name/address/phone different across sites
   - Fix: Audit all citations, standardize everywhere

7. **Broken links (404s)**
   - Problem: Pages deleted without redirects
   - Fix: Check GSC for crawl errors, 301 redirect to new page

8. **Ignoring mobile**
   - Problem: Mobile is 70% of searches; yours is broken
   - Fix: Responsive design, test regularly, prioritize mobile in design

9. **No structured data**
   - Problem: Google doesn't understand what you are
   - Fix: Add LocalBusiness + Service + Review schemas

10. **Zero backlinks**
    - Problem: No external sites link to you = no authority
    - Fix: Create great content, reach out to industry partners, get reviewed in directories

---

## Implementation Timeline

**Month 1: Foundation**
- [ ] Claim/optimize Google Business Profile
- [ ] Audit NAP consistency across web
- [ ] Set up Google Search Console
- [ ] Add LocalBusiness schema to homepage
- [ ] Write/optimize homepage and service pages
- [ ] Improve page titles and meta descriptions

**Month 2: Content & Structure**
- [ ] Create service area pages (if multi-location)
- [ ] Build content cluster (1 hub + 3 cluster pages)
- [ ] Add FAQ schema
- [ ] Implement internal linking strategy
- [ ] Set up review request process

**Month 3: Authority**
- [ ] Launch review campaign (get 20+ reviews)
- [ ] Add review schema to homepage
- [ ] Create high-value backlinks (directory listings, local partnerships)
- [ ] Monitor Google Search Console weekly
- [ ] Fix any indexation issues

**Ongoing (Monthly):**
- [ ] Monitor Search Console (1 hour)
- [ ] Publish 1-2 blog posts
- [ ] Request 5-10 reviews
- [ ] Respond to all reviews
- [ ] Check page speed (target: 80+ on PageSpeed)
- [ ] Update Google Business Profile photos/info

---

## Tools You Actually Need

| Tool | Cost | Purpose |
|------|------|---------|
| Google Business Profile | Free | Local search + Google Maps presence |
| Google Search Console | Free | Monitor, debug, improve SEO |
| Google PageSpeed Insights | Free | Check site speed, mobile usability |
| Schema.org | Free | Learn and validate structured data |
| Yoast SEO or Rank Math | Free-$99/yr | Title, description, readability optimization |
| Semrush or Ahrefs | $99-399/mo | Keyword research, competitor analysis |
| Screaming Frog | Free-$149 | Crawl your site, find technical issues |

---

## Measuring Success

Don't just guess. Measure.

**Metrics that matter:**

1. **Google Business Profile views** (target: +50% in 6 months)
2. **Organic search traffic** (target: +100% in 12 months)
3. **Local search keywords ranking** (track top 20 keywords)
4. **Click-through rate from search** (target: 3%+ improvement)
5. **Online reviews count + rating** (target: +2-3 new reviews/month)
6. **Service area ranking** (are you ranking for "[service] in [city]"?)

**Review every month in Google Search Console. Adjust strategy based on data, not guesses.**

---

## Final Thoughts

SEO isn't magic. It's not luck. It's a system.

You control:
- Your Google Business Profile (100%)
- Your website content (100%)
- Your technical foundation (100%)
- Your review request process (100%)
- Your internal linking (100%)

You influence:
- Review reputation (70% — you can request and respond, but customers choose what they write)
- Backlinks (50% — you can create great content, but others choose to link)
- Rankings (depends on competition, but fundamentals matter more than luck)

Stop waiting. Implement ONE thing this week. Then another. In 6 months, you'll be searching for yourself and seeing yourself at the top.

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
