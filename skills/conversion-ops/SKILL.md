---
name: conversion-ops
description: >
  Comprehensive conversion optimization, analytics, and growth engineering patterns for modern web apps. Master Google Analytics 4 setup with Next.js, event tracking architecture, A/B testing with Vercel Edge Config, funnel analysis, landing page optimization, onboarding flows, retention patterns, pricing strategies, KPI frameworks, and internal analytics dashboards. Covers acquisition metrics (CAC, traffic sources), activation (signup/onboarding completion), revenue (MRR, ARPU, LTV), and retention (churn prediction, DAU/MAU). Includes event taxonomy, statistical significance, cohort analysis, and real-world conversion optimization tactics.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Google Analytics 4, conversion optimization, growth engineering patterns"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Conversion Operations

The difference between a visitor and a paying customer is measurable. This skill teaches you to track it, optimize it, and scale it.

## 1. Analytics Setup: Google Analytics 4 + Next.js

### Core Configuration
```typescript
// app/layout.tsx — GA4 with gtag
import Script from 'next/script';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <head>
        <Script
          strategy="afterInteractive"
          src={`https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX`}
        />
        <Script
          id="gtag-init"
          strategy="afterInteractive"
          dangerouslySetInnerHTML={{
            __html: `
              window.dataLayer = window.dataLayer || [];
              function gtag(){dataLayer.push(arguments);}
              gtag('js', new Date());
              gtag('config', 'G-XXXXXXXXXX', {
                send_page_view: true,
                anonymize_ip: true,
              });
            `,
          }}
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### Server-Side Event Tracking
```typescript
// lib/events.ts — Supabase + GA4 integration
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(url, key);

export async function trackEvent(
  userId: string,
  eventName: string,
  eventParams: Record<string, any>,
) {
  // Log to Supabase for internal analytics
  await supabase.from('events').insert({
    user_id: userId,
    event_name: eventName,
    event_params: eventParams,
    timestamp: new Date(),
  });

  // Send to GA4 via server-side endpoint
  await fetch('https://www.google-analytics.com/mp/collect', {
    method: 'POST',
    body: JSON.stringify({
      measurement_id: process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID,
      api_secret: process.env.GA_SECRET,
      events: [
        {
          name: eventName,
          params: {
            ...eventParams,
            session_id: userId,
            timestamp_micros: Date.now() * 1000,
          },
        },
      ],
    }),
  });
}
```

### Custom Dimensions & User Properties
```typescript
// Register user properties in GA4
gtag('set', {
  'user_id': userId,
  'custom_map': {
    'dimension1': 'subscription_tier',
    'dimension2': 'signup_source',
    'dimension3': 'feature_flags',
  },
});

gtag('event', 'user_engagement', {
  'subscription_tier': 'pro',
  'signup_source': 'organic_search',
  'feature_flags': 'variant_a',
});
```

## 2. Event Tracking Architecture

### Event Naming Convention
Follow the pattern: `object_action` (lowercase, snake_case)
- `user_signup_started` → User clicks signup form
- `user_signup_completed` → Form submission successful
- `pricing_plan_viewed` → User views pricing page
- `plan_purchase_initiated` → Checkout begun
- `plan_purchase_completed` → Transaction successful
- `feature_used_export` → User exports data
- `notification_email_opened` → Email tracking
- `onboarding_step_1_completed` → Onboarding progression

### Event Taxonomy Template
```typescript
interface EventPayload {
  event_name: string;
  user_id: string;
  session_id: string;
  timestamp: number;

  // Context
  page_path: string;
  page_title: string;
  referrer: string;

  // Event-specific
  event_value?: number;
  event_category?: string;
  element_id?: string;

  // User cohort
  is_new_user: boolean;
  days_since_signup?: number;
  subscription_tier?: string;

  // Funnel position
  funnel_name?: string;
  funnel_step?: number;
}
```

### Tracking Plan
```
Signup Funnel:
├─ landing_page_viewed (traffic source, utm params)
├─ signup_form_started (auto-fill detected)
├─ signup_form_field_completed (field name, time)
├─ signup_form_submitted (form duration)
├─ signup_email_verification_sent
├─ signup_email_verified
└─ signup_completed (total funnel duration)

Activation Funnel:
├─ onboarding_started
├─ onboarding_step_1_completed (profile setup)
├─ onboarding_step_2_completed (first integration)
├─ onboarding_step_3_completed (first sync)
├─ activation_checklist_viewed
├─ first_real_action (import/sync/export)
└─ activation_completed

Subscription Funnel:
├─ pricing_page_viewed (traffic source)
├─ plan_selected (which plan)
├─ checkout_initiated (plan, discount)
├─ checkout_step_1_completed (payment info)
├─ checkout_error (error code)
└─ purchase_completed (amount, plan, currency)
```

## 3. A/B Testing: Feature Flags + Vercel Edge Config

### Setup with Vercel Edge Config
```typescript
// lib/experiments.ts
import { getEdgeConfig } from '@vercel/edge-config';

interface Experiment {
  name: string;
  variants: Record<string, number>; // variant: weight
  startDate: number;
  endDate: number;
}

export async function getVariant(
  experimentName: string,
  userId: string,
): Promise<string> {
  const config = await getEdgeConfig();
  const experiment = config.get<Experiment>(experimentName);

  if (!experiment) return 'control';
  if (Date.now() < experiment.startDate) return 'control';
  if (Date.now() > experiment.endDate) return 'control';

  // Deterministic assignment: same user always gets same variant
  const hash = hashUserId(userId);
  const rand = (hash % 100) / 100;

  let cumulative = 0;
  for (const [variant, weight] of Object.entries(experiment.variants)) {
    cumulative += weight;
    if (rand <= cumulative / 100) return variant;
  }
  return 'control';
}
```

### Client-Side Implementation
```typescript
// app/pricing/page.tsx
'use client';

import { useEffect, useState } from 'react';
import { trackEvent } from '@/lib/events';

export default function PricingPage() {
  const [variant, setVariant] = useState<string | null>(null);

  useEffect(() => {
    (async () => {
      const v = await fetch(`/api/experiment?name=pricing_cta_text&userId=${userId}`).then(r => r.json());
      setVariant(v.variant);

      // Track which variant user saw
      trackEvent(userId, 'pricing_variant_assigned', {
        experiment: 'pricing_cta_text',
        variant: v.variant,
      });
    })();
  }, []);

  const ctaText = variant === 'variant_a' ? 'Start Free Trial' : 'Get Started Now';

  return (
    <button
      onClick={() => {
        trackEvent(userId, 'pricing_cta_clicked', {
          variant,
          cta_text: ctaText,
        });
      }}
    >
      {ctaText}
    </button>
  );
}
```

### Statistical Significance
For 80% power, 95% confidence, and 20% lift detection:
- Sample size per variant: n = ((1.96 + 0.84) / 0.20)² × 0.5 × 0.5 = ~785 per variant
- **Minimum test duration**: 1-2 weeks (seasonal variations matter)
- **Stop test when**: Variant is >95% likely better OR you hit predetermined sample size
- **Never peek**: Calculate sample size upfront; don't stop early on false positives

## 4. Funnel Analysis

### Define Conversion Funnels
```
Signup Funnel:
Step 1: Landing page view (traffic baseline)
Step 2: Signup form interaction (engagement)
Step 3: Form submission (commitment)
Step 4: Email verification (identity confirmation)
Drop-off analysis: Which step loses most users?
```

```
Onboarding Funnel:
Step 1: Profile setup completed (basic info)
Step 2: First integration connected (proof of value)
Step 3: First sync/data imported (tangible result)
Step 4: Feature usage (activation)
```

### Funnel Visualization Query (SQL)
```sql
WITH funnel_steps AS (
  SELECT
    user_id,
    event_name,
    MAX(CASE WHEN event_name = 'signup_form_started' THEN timestamp END) as step_1,
    MAX(CASE WHEN event_name = 'signup_form_submitted' THEN timestamp END) as step_2,
    MAX(CASE WHEN event_name = 'signup_email_verified' THEN timestamp END) as step_3,
    MAX(CASE WHEN event_name = 'activation_completed' THEN timestamp END) as step_4
  FROM events
  WHERE created_at > NOW() - INTERVAL '30 days'
  GROUP BY user_id
)
SELECT
  COUNT(DISTINCT user_id) as started,
  COUNT(DISTINCT CASE WHEN step_2 IS NOT NULL THEN user_id END) as submitted,
  COUNT(DISTINCT CASE WHEN step_3 IS NOT NULL THEN user_id END) as verified,
  COUNT(DISTINCT CASE WHEN step_4 IS NOT NULL THEN user_id END) as activated,
  ROUND(100.0 * COUNT(DISTINCT CASE WHEN step_2 IS NOT NULL THEN user_id END) / COUNT(DISTINCT user_id), 2) as step_1_to_2_rate,
  ROUND(100.0 * COUNT(DISTINCT CASE WHEN step_4 IS NOT NULL THEN user_id END) / COUNT(DISTINCT user_id), 2) as overall_conversion_rate
FROM funnel_steps;
```

### Cohort Analysis
```sql
SELECT
  DATE_TRUNC('week', created_at) as signup_week,
  DATEDIFF(day, created_at, MAX(CASE WHEN event_name = 'plan_purchase_completed' THEN timestamp END)) as days_to_conversion,
  COUNT(DISTINCT user_id) as users_converted,
  ROUND(100.0 * COUNT(DISTINCT user_id) / LAG(COUNT(DISTINCT user_id)) OVER (ORDER BY DATE_TRUNC('week', created_at)), 2) as conversion_rate
FROM users
LEFT JOIN events ON users.id = events.user_id
WHERE users.created_at > NOW() - INTERVAL '90 days'
GROUP BY DATE_TRUNC('week', created_at)
ORDER BY signup_week DESC;
```

## 5. Landing Page Optimization

### Above-Fold Checklist
- **Headline**: Job-to-be-done focused ("Get X done 10x faster")
- **Subheadline**: Specific benefit ("No credit card. 3-minute setup.")
- **Primary CTA**: Clear action ("Start Free Trial")
- **Social proof**: Customer count or logo ("{X}+ teams trust us")
- **Hero image**: Shows product in context, not generic
- **Form fields**: Minimum viable fields (name, email, password only)

### CTA Optimization
```typescript
// Test variants
const ctaVariants = {
  variant_a: {
    text: 'Start Free Trial',
    color: '#007BFF',
    size: 'large',
  },
  variant_b: {
    text: 'Get Started — No CC Required',
    color: '#28A745',
    size: 'large',
  },
  variant_c: {
    text: 'Try for Free',
    color: '#007BFF',
    size: 'medium',
  },
};

// Winner: Variant B typically converts 15-25% better
// Key factor: "No CC Required" removes friction
```

### Form Optimization
- **Single-step form**: First time only (email + password)
- **Progressive disclosure**: Ask for billing info during checkout, not signup
- **Autofill hints**: `autocomplete="email"`, `autocomplete="given-name"`
- **Field validation**: Real-time, show errors inline
- **Reduce fields**: Each field costs 1-3% conversion rate

### Page Speed Impact
- Load time 0→1s: 7% conversion drop per 100ms delay
- Measure: Largest Contentful Paint (LCP) < 2.5s
- Optimize: Image optimization, font subsetting, code splitting

## 6. Onboarding Optimization

### First-Run Experience
```typescript
// components/OnboardingFlow.tsx
export function OnboardingFlow({ userId }: { userId: string }) {
  const steps = [
    { id: 'profile', label: 'Tell us about you', duration: '2 min' },
    { id: 'integration', label: 'Connect your tool', duration: '3 min' },
    { id: 'first_sync', label: 'Import your data', duration: '1 min' },
  ];

  return (
    <div className="flex gap-8">
      {/* Checklist sidebar */}
      <div className="w-48">
        {steps.map((step) => (
          <div
            key={step.id}
            className={`p-3 rounded ${completedSteps.includes(step.id) ? 'bg-green-100' : 'bg-gray-100'}`}
          >
            <div className="font-medium">{step.label}</div>
            <div className="text-sm text-gray-500">{step.duration}</div>
          </div>
        ))}
      </div>

      {/* Main content */}
      <div className="flex-1">
        {currentStep === 'profile' && <ProfileSetup />}
        {currentStep === 'integration' && <IntegrationConnector />}
        {currentStep === 'first_sync' && <FirstSyncStatus />}
      </div>
    </div>
  );
}
```

### Progressive Disclosure
- **Step 1 (Profile)**: Name, email, company size → Estimate time-to-value
- **Step 2 (Integration)**: Connect source (Salesforce, HubSpot, etc.) → Show value
- **Step 3 (First Sync)**: Run first import → Tangible proof
- **Step 4 (Feature Tour)**: Show key features → Build confidence
- **Step 5 (Prompt to Action)**: Ask for first real action → Activation

### Activation Metrics
```typescript
// lib/activation.ts
export async function calculateActivationScore(userId: string) {
  const user = await supabase
    .from('users')
    .select('created_at')
    .eq('id', userId)
    .single();

  const events = await supabase
    .from('events')
    .select('event_name')
    .eq('user_id', userId)
    .gte('timestamp', user.created_at)
    .lte('timestamp', user.created_at + 7 * 24 * 60 * 60 * 1000);

  const activationEvents = [
    'onboarding_step_1_completed',
    'onboarding_step_2_completed',
    'first_real_action', // import/sync/export
  ];

  const score = events.filter(e => activationEvents.includes(e.event_name)).length;
  return score >= 2 ? 'activated' : 'pending';
}
```

## 7. Retention Patterns

### Engagement Loops
Email notifications (new data available) → User opens app → Views results → Takes action → More data collected → Repeat

### Notification Strategy
- **Frequency**: 1-2 emails per week max (avoid fatigue)
- **Trigger**: User action results ("Your report is ready")
- **Segmentation**: Long-inactive users get re-engagement emails
- **Unsubscribe**: Always honor, but try save with weekly digest option

### Churn Prediction Signals
```sql
SELECT
  user_id,
  DATEDIFF(day, MAX(last_login), NOW()) as days_inactive,
  COUNT(DISTINCT DATE(event_timestamp)) as active_days_last_30,
  COUNT(DISTINCT CASE WHEN event_name LIKE '%feature%' THEN event_timestamp END) as feature_usage_events
FROM events
WHERE created_at > NOW() - INTERVAL '30 days'
GROUP BY user_id
HAVING DATEDIFF(day, MAX(last_login), NOW()) > 14
  AND active_days_last_30 < 5;
```

Risk tiers:
- **Low risk**: Active last 7 days, 10+ feature events/month
- **Medium risk**: Active last 14 days, 5-10 feature events/month
- **High risk**: Inactive >14 days, <5 feature events/month → Send re-engagement email

### NPS/CSAT Tracking
```typescript
// components/SurveyPrompt.tsx
export function SurveyPrompt({ userId }: { userId: string }) {
  const handleNPS = (score: number) => {
    trackEvent(userId, 'nps_response', {
      score,
      product_area: 'general',
      days_since_signup: daysActive,
    });
  };

  return (
    <div className="p-4 bg-blue-50 rounded">
      <p className="font-medium mb-3">How likely to recommend us? (0-10)</p>
      <div className="flex gap-2">
        {[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10].map((score) => (
          <button key={score} onClick={() => handleNPS(score)}>
            {score}
          </button>
        ))}
      </div>
    </div>
  );
}
```

## 8. Pricing Page Patterns

### Pricing Table Layout
```
┌─────────────────────────────────────────────────┐
│        Starter      Professional      Enterprise│
│ Price   $29/mo   →    $99/mo    (Recommended)   │
│ Users      2            Unlimited               │
│ API       No            Yes                     │
│ Support  Email    Chat + Phone                  │
│ CTA    Choose      Get Started      Contact Us  │
└─────────────────────────────────────────────────┘
```

### Anchor Pricing
- **Mid-tier recommended**: Highlight with badge, border, shadow
- **Price anchoring**: High tier justifies mid-tier value
- **Toggle annual/monthly**: 20% discount for annual (5-10% typically)

### Social Proof Near CTA
```
"{X}+ companies trust us for their {job-to-be-done}"
[Logo Logo Logo Logo]
"4.9★ on G2 — See reviews"
```

## 9. Performance Metrics: KPI Framework

### Acquisition (Top of Funnel)
- **CAC** (Customer Acquisition Cost): Total marketing spend / new customers acquired
- **Traffic sources**: Direct, organic, paid, referral breakdown
- **Conversion rate by source**: Which channels bring quality customers?
- **Target**: CAC < 3x monthly subscription value

### Activation (Funnel Conversion)
- **Signup-to-activation rate**: % of signups completing onboarding
- **Onboarding completion time**: Days from signup to first real action
- **Target**: 40-60% complete onboarding within 14 days

### Revenue (Monetization)
- **MRR** (Monthly Recurring Revenue): Sum of all active subscriptions
- **ARPU** (Average Revenue Per User): MRR / total active users
- **LTV** (Lifetime Value): ARPU × average subscription duration
- **Target**: LTV:CAC ratio > 3:1

### Retention (Long-term Health)
- **Churn rate**: (Customers lost / customers start of month) × 100
- **DAU/MAU**: Daily active users / monthly active users (engagement ratio)
- **Target**: <5% monthly churn, DAU/MAU > 30%

## 10. Internal Analytics Dashboard

### Key Dashboard Sections
```
┌────────────────────────────────────────┐
│ Monthly Overview                       │
│ MRR: $50,200 ↑ 12% | Churn: 3.2% ↓   │
│ New customers: 42 | LTV: $4,800       │
└────────────────────────────────────────┘

┌──────────────┬──────────────┬──────────────┐
│ Signup Funnel│ Onboard Rate │ Revenue Trend│
│   1,250 → 487│   39% ↑      │    Chart    │
│   39% overall│   +2% vs mo  │   30-day    │
└──────────────┴──────────────┴──────────────┘

┌──────────────┬──────────────┐
│ Churn Risk   │ Traffic Mix  │
│ High: 12     │ Organic: 45% │
│ Medium: 28   │ Paid: 35%    │
│ Low: 185     │ Referral: 20%│
└──────────────┴──────────────┘
```

### Core Dashboard Queries
```typescript
// Dashboard data fetching
async function getDashboardMetrics(startDate: Date, endDate: Date) {
  const metrics = {
    // Revenue
    mrr: await supabase.rpc('calculate_mrr', { start: startDate, end: endDate }),

    // Funnel
    signup_count: await supabase.from('events').countEq('event_name', 'signup_completed'),
    onboard_count: await supabase.from('events').countEq('event_name', 'activation_completed'),

    // Retention
    churn_rate: await supabase.rpc('calculate_churn_rate', { end: endDate }),

    // Trend
    mrr_previous_month: await supabase.rpc('calculate_mrr', {
      start: new Date(startDate.getTime() - 30 * 24 * 60 * 60 * 1000),
      end: startDate,
    }),
  };

  return metrics;
}
```

### Time Period Selector
```typescript
const periods = [
  { label: '7 days', days: 7 },
  { label: '30 days', days: 30 },
  { label: '90 days', days: 90 },
  { label: 'YTD', custom: 'year_to_date' },
];

// Each changes date range, refetches metrics, shows trend vs previous period
```

## Measurement Maturity Roadmap

**Phase 1 (Week 1-2)**: GA4 setup, event tracking, signup funnel
**Phase 2 (Week 3-4)**: Onboarding funnel, activation metrics, cohort analysis
**Phase 3 (Month 2)**: A/B testing, CAC/LTV tracking, pricing optimization
**Phase 4 (Month 3)**: Predictive churn, dashboard automation, growth loops

Start with signup funnel. Everything else flows from converting visitors to paying customers.

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
