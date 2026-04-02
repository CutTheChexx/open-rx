<!-- Open RX — Created by CutTheChexx. All rights reserved. -->
# Diagnostic Playbook — Question Patterns by Project Type

When a patient walks in, their first few words tell you what kind of project this is.
Use the right diagnostic track to ask the right questions fast.

---

## Track 1: Dashboard / Tracker / Internal Tool

**Patient says things like:** "I need a dashboard", "track my leads", "monitor bids",
"show me my pipeline", "team needs to see X"

**Fast diagnostic:**
1. What data are you tracking? (leads, bids, tasks, inventory, etc.)
2. Where does the data live today? (spreadsheet, head, existing DB, nowhere)
3. Who needs to see it? (just you, your team of [N], external clients)
4. What's the #1 thing you do when you open it? (the primary action)
5. Any automations? (daily scrapes, notifications, scoring, emails)

**Common features to probe for:**
- Filtering and sorting (by status, date, priority, assignee)
- Search
- Status workflow (hot → warm → closed → archived)
- KPI cards at the top (total count, conversion rate, total value)
- Export (CSV, PDF, share link)
- Activity log (who did what when)

**RX typically includes:** Supabase table, React dashboard, tab navigation,
filter bar, status badges, KPI cards, maybe a sidebar.

---

## Track 2: Landing Page / Marketing Site

**Patient says things like:** "I need a landing page", "website for my business",
"something to show clients", "we need an online presence"

**Fast diagnostic:**
1. What's the ONE thing you want visitors to do? (call, fill form, book, buy)
2. Who's visiting? (homeowners, businesses, investors, general public)
3. Do you have a logo, brand colors, or existing brand materials?
4. Any existing site? (redesign vs. brand new)
5. Need a form? What info are you collecting?

**Common features to probe for:**
- Hero section with CTA
- Services/offerings section
- Social proof (reviews, testimonials, logos, case studies)
- Contact form or booking widget
- SEO (local? what keywords?)
- Analytics (Google Analytics, conversion tracking)
- Mobile responsiveness (usually 60%+ traffic on mobile for local biz)

**RX typically includes:** Single-page or multi-page HTML/React, hero + CTA,
services grid, testimonials, contact form, JSON-LD structured data, meta tags.

---

## Track 3: Automation / Scheduled Task / Pipeline

**Patient says things like:** "I want to automate", "run this every day",
"scrape this site", "send me alerts when", "connect X to Y"

**Fast diagnostic:**
1. What triggers it? (time-based schedule, event, manual button)
2. What does it DO? (fetch data, transform, send notification, write to DB)
3. Where does the output go? (email, database, Slack, file, dashboard)
4. How often? (every hour, daily, weekly, on-demand)
5. What happens if it fails? (retry, alert, skip)

**Common features to probe for:**
- Data source authentication (API keys, login, public)
- Rate limiting / politeness (don't hammer APIs)
- Error handling and retry logic
- Logging (know when it ran and what happened)
- Output format (structured data, email digest, dashboard feed)
- Manual trigger option (run on-demand, not just scheduled)

**RX typically includes:** Edge Function or scheduled task, data parsing logic,
database writes, error handling, notification on failure, optional dashboard view.

---

## Track 4: Full SaaS / Multi-Feature App

**Patient says things like:** "I'm building a platform", "users need accounts",
"subscription-based", "I want to build something like [established product]"

**Fast diagnostic:**
1. In one sentence, what does a user get from this? (the value prop)
2. What's the user journey? (sign up → onboard → core action → repeat)
3. How do you make money? (subscription, one-time, freemium, usage-based)
4. What's the MVP? (if you could only ship 3 features, which 3?)
5. Who's your competition? (what exists that's close to this?)

**Common features to probe for:**
- Auth (sign up, login, password reset, OAuth)
- Onboarding flow (first-run experience)
- Billing (Stripe, plans, upgrade/downgrade)
- Admin panel (manage users, view metrics)
- Settings (profile, notifications, team management)
- Multi-tenancy (organizations, teams, workspaces)
- Roles and permissions

**RX typically includes:** Multi-part RX series. Part 1 = auth + core feature + database.
Part 2 = secondary features + billing. Part 3 = admin + polish.
Always warn: "This is a multi-session build."

---

## Track 5: API / Integration / Backend Service

**Patient says things like:** "I need an API", "webhook handler", "edge function",
"connect my app to X", "data pipeline"

**Fast diagnostic:**
1. What consumes this API? (frontend, mobile app, third-party, another service)
2. What operations does it need? (CRUD? Transform? Aggregate? Proxy?)
3. Auth? (API key, JWT, public, service-to-service)
4. What's the data format? (JSON, CSV, XML, binary)
5. Volume? (10 requests/day or 10,000/second)

**Common features to probe for:**
- Input validation (Zod, JSON Schema)
- Rate limiting
- Error response format (consistent error codes)
- Logging and monitoring
- Versioning (v1/v2 endpoints)
- Documentation (OpenAPI/Swagger)

**RX typically includes:** Edge Function or API route specs, request/response schemas,
auth middleware, error handling patterns, example curl commands.

---

## Track 6: Component / Feature Addition (Patch)

**Patient says things like:** "add a feature to my existing app", "I want a new tab",
"can we add X to the dashboard", "extend this with Y"

**Fast diagnostic:**
1. What's the existing app? (name, tech stack, where it lives)
2. What are you adding? (new tab, new section, new data source, new automation)
3. Does it need new data? (new table, new columns, new API)
4. Does it change existing behavior? (or is it purely additive)
5. Where does it plug in? (which screen, which component tree)

**Common features to probe for:**
- Impact on existing features (does anything break or change?)
- New database tables or columns
- New navigation items (tab, sidebar link, menu item)
- State management changes
- Performance impact (adding a heavy feature to a fast page?)

**RX uses Patch format:** Not a full RX — a focused patch document that says
"stack this on top of [existing RX]."

---

## Universal Follow-Up Questions

These work for ANY project type. Use when you need to go deeper:

**Clarification:**
- "When you say [X], what does that look like in practice?"
- "Walk me through a typical day using this — what's the first thing you do?"
- "What's the most annoying part of how you do this today?"

**Priority:**
- "If you could only have ONE of those features on day one, which one?"
- "What would make you say 'this is done enough to use'?"
- "What's the thing that, if it's missing, makes this useless?"

**Edge cases:**
- "What happens when there's no data yet? Like day one, empty?"
- "What if two people edit the same thing at the same time?"
- "What's the biggest dataset this needs to handle?"

**Taste:**
- "Show me something you think looks good — a site, an app, anything"
- "What do you hate about tools you've used before?"
- "Dark mode or light mode person?"

---

## Red Flags to Watch For

| Red Flag | What to Do |
|----------|-----------|
| "I want it to do EVERYTHING" | Break into multi-part RX. Scope the MVP first. |
| "Just make it like [massive platform]" | Ask: "What's the ONE thing about [platform] you want?" |
| Changing requirements mid-consultation | Pause. Re-confirm the diagnosis. Write down what changed. |
| "I don't care how it looks" | They care. They just can't articulate it. Show them 2-3 options. |
| No clear user in mind | Probe: "Who opens this app on Monday morning? What do they need?" |
| "Can you add AI to it?" | Ask: "What decision or task do you want AI to help with specifically?" |
| Scope creep (feature list keeps growing) | Draw the line: "That's great for v2. Let's lock v1 at these [N] features." |

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
