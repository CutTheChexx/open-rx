<!-- Open RX — Created by CutTheChexx. All rights reserved. -->
# RX Template — Full Prescription (Copy & Fill)

Use this template when writing a complete RX from a consultation.
Replace everything in `[brackets]` with project-specific content.
Delete any sections that don't apply (e.g., no API section if it's a static site).

---

# [Project Name] — RX (Prescription)
**Version:** 1.0
**Written:** [YYYY-MM-DD]
**Patient:** [Name / team / company]
**Diagnosis:** [The problem this solves — 1-2 plain-English sentences]

---

## 1. WHAT IT IS

[2-3 sentences. Non-technical. Answer: "What does this thing do and why does it exist?"
Example: "A dashboard that pulls leads from three different sources, scores them
based on how well they match the company's ideal customer, and surfaces the hottest
opportunities so the sales team can close faster."]

---

## 2. WHO USES IT

| Role | What They Do | Access Level |
|------|-------------|-------------|
| [Role 1] | [Primary actions] | [Admin / Editor / Viewer] |
| [Role 2] | [Primary actions] | [Access level] |

**Total expected users:** [number or range]
**Auth required:** [Yes/No — if yes, specify method]

---

## 3. TECH STACK

| Layer | Choice | Why |
|-------|--------|-----|
| Frontend | [React / Next.js / static HTML / etc.] | [1-line reason] |
| Styling | [Tailwind + shadcn / custom CSS / etc.] | [1-line reason] |
| Backend | [Supabase / Edge Functions / Express / none] | [1-line reason] |
| Database | [Supabase Postgres / SQLite / none] | [1-line reason] |
| Auth | [Supabase Auth / none / API key] | [1-line reason] |
| Hosting | [Vercel / Netlify / Supabase / static] | [1-line reason] |

---

## 4. DATA MODEL

```sql
-- [Table 1 Name]
CREATE TABLE [table_name] (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  [column_name] [TYPE] [CONSTRAINTS],
  [column_name] [TYPE] [CONSTRAINTS],
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Indexes
CREATE INDEX idx_[name] ON [table]([column]);

-- RLS (if applicable)
ALTER TABLE [table] ENABLE ROW LEVEL SECURITY;
CREATE POLICY "[description]" ON [table] USING ([condition]) WITH CHECK ([condition]);
```

[Repeat for each table. Include relationships, foreign keys, and explain
what each table stores in a comment above the CREATE statement.]

**Entity Relationship Summary:**
```
[Table A] 1──▶ N [Table B] (one-to-many via foreign key)
[Table B] N──▶ 1 [Table C]
```

---

## 5. CORE FEATURES (Build Order)

Features listed in the order they should be built. Each one includes
what it does, what the user sees, and what data it touches.

### 5.1 [Feature Name]
**What it does:** [Plain English — 1-2 sentences]
**User sees:**
- [UI element 1 — e.g., "A table showing all active bids sorted by score"]
- [UI element 2 — e.g., "Filter bar with status, category, and date range"]
- [UI element 3]
**Data:** Reads from `[table]`, writes to `[table]`
**Key interactions:**
- [Click X → happens Y]
- [Filter by Z → table updates]

### 5.2 [Feature Name]
[Same structure]

### 5.3 [Feature Name]
[Same structure]

[Continue for all features. Typical RX has 3-8 features.]

---

## 6. SCREEN-BY-SCREEN LAYOUT

### Screen 1: [Name] (e.g., "Main Dashboard")
```
┌─────────────────────────────────────────────┐
│  [Header / Nav Bar]                         │
├─────────────────────────────────────────────┤
│  [KPI Cards Row]                            │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ KPI1 │ │ KPI2 │ │ KPI3 │ │ KPI4 │       │
│  └──────┘ └──────┘ └──────┘ └──────┘       │
├─────────────────────────────────────────────┤
│  [Main Content Area]                        │
│  - [Component A]                            │
│  - [Component B]                            │
├─────────────────────────────────────────────┤
│  [Footer / Status Bar]                      │
└─────────────────────────────────────────────┘
```
**Components on this screen:**
- [Component name] — [what it shows/does]
- [Component name] — [what it shows/does]

**Responsive:** [How it stacks on mobile — e.g., "KPI cards stack 2x2, table becomes card list"]

### Screen 2: [Name]
[Same structure]

---

## 7. VISUAL SPECS

| Element | Style |
|---------|-------|
| Page background | `[hex]` |
| Card background | `[hex]` |
| Card border | `[radius, color, width]` |
| Primary accent | `[hex]` — used for [buttons, badges, etc.] |
| Secondary accent | `[hex]` — used for [links, hover states, etc.] |
| Text primary | `[hex]` |
| Text secondary | `[hex]` |
| Font | `[font-family]` |
| Border radius | `[px]` |
| Spacing system | `[e.g., 4px base, multiples of 4]` |

**Theme:** [Dark / Light / Both]
**Animations:** [e.g., "Cards fade-in on load, hover lifts 2px with shadow"]
**Brand assets:** [Logo URL, brand guide reference if any]

---

## 8. API / INTEGRATIONS

### [Integration Name] (e.g., "Scrape Edge Function")
- **Type:** [Edge Function / Webhook / REST API / Cron Job]
- **Trigger:** [How/when it runs — e.g., "pg_cron every 6 hours"]
- **Input:** [What it receives]
- **Processing:** [What it does, step by step]
- **Output:** [What it writes/returns]
- **Error handling:** [What happens on failure]

### [Integration Name]
[Same structure]

---

## 9. EDGE CASES & ERROR HANDLING

| Scenario | Handling |
|----------|----------|
| Empty data (no records yet) | [Show empty state with message and CTA — e.g., "No bids found. Check back tomorrow."] |
| API/fetch failure | [Show error toast, retry button, log to console] |
| Slow network | [Loading skeletons, not spinners] |
| Invalid user input | [Inline validation messages, prevent submission] |
| [Project-specific edge case] | [How to handle it] |

---

## 10. ANTI-PATTERNS (Don't Do This)

- **Don't use lorem ipsum** — Use realistic sample data that matches the domain
- **Don't center-align everything** — Left-align content, center only heroes/CTAs
- **Don't forget empty states** — Every list/table needs a "nothing here yet" view
- **Don't hardcode URLs or keys** — Environment variables for everything
- **Don't skip mobile** — Test at 375px width
- [Project-specific anti-patterns]

---

## 11. SUCCESS CRITERIA

- [ ] [Core feature 1] works end-to-end with real data
- [ ] [Core feature 2] works end-to-end
- [ ] Loads in under [X] seconds on desktop
- [ ] Responsive at 375px, 768px, 1024px, 1440px
- [ ] All error states handled gracefully
- [ ] Sample data seeded and realistic
- [ ] [Project-specific success metric]

---

## 12. BUILD ORDER

Execute in this sequence:

1. **Database** — Create tables, indexes, RLS policies, seed sample data
2. **Data layer** — Supabase client setup, fetch functions, types
3. **[Core screen/feature]** — Build the main view first
4. **[Supporting feature]** — Add filters, search, sorting
5. **[Secondary feature]** — Build the next screen/tab
6. **[Integrations]** — Edge functions, cron jobs, external APIs
7. **[Polish]** — Animations, responsive tweaks, empty states, error handling
8. **[Verify]** — Test against success criteria, fix any gaps

---

## COMPONENT TREE

```
[AppRoot]
├── [Layout/Shell]
│   ├── [Header]
│   │   ├── [Logo]
│   │   ├── [Nav]
│   │   └── [UserMenu]
│   ├── [MainContent]
│   │   ├── [Feature1Component]
│   │   ├── [Feature2Component]
│   │   └── [Feature3Component]
│   └── [Footer/StatusBar]
```

---

*End of prescription. Fill this RX by building in the order specified above.*

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
