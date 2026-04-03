---
name: ship-it
description: >
  Production deployment automation for Next.js apps. Covers Vercel deployments, GitHub Actions CI/CD pipelines, environment management, Docker containerization, database migrations, preview environments, monitoring, and rollback strategies. Deploy with confidence using this complete guide to shipping code from laptop to production at scale.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Vercel docs, GitHub Actions, Docker, Supabase, deployment best practices"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## Deployment Architecture Overview

Deploying production applications requires coordination across multiple systems: version control, CI/CD pipelines, container orchestration, database management, and monitoring. This guide covers the complete pipeline from local development to scaled production environments.

## 1. Vercel Deployment

Vercel is the optimal platform for Next.js applications, providing git integration, automatic deployments, edge functions, and serverless scaling.

### Project Setup

```bash
# Install Vercel CLI
npm install -g vercel

# Login and link project
vercel login
vercel link

# Deploy preview
vercel --prod false

# Deploy to production
vercel --prod
```

### Vercel Configuration (vercel.json)

```json
{
  "buildCommand": "next build",
  "devCommand": "next dev",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "nodeVersion": "18.x",
  "env": [
    "NEXT_PUBLIC_API_URL",
    "DATABASE_URL",
    "EDGE_CONFIG"
  ],
  "envPrefix": "VERCEL_",
  "regions": ["sfo1", "iad1", "lhr1"],
  "functions": {
    "api/**": {
      "memory": 1024,
      "maxDuration": 60
    },
    "pages/**": {
      "memory": 512,
      "maxDuration": 30
    }
  }
}
```

### Environment Variables

```bash
# .env.local (local development)
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=postgresql://user:pass@localhost/dev
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=xxx

# Vercel UI → Settings → Environment Variables
# Set for: Production, Preview, Development
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
DATABASE_URL=postgresql://user:pass@db.yourdomain.com/prod
EDGE_CONFIG=https://edge-config.vercel.sh/xxx
```

### Preview Deployments

Every push to a feature branch creates an automatic preview deployment:
- Unique preview URL: `https://<branch>.<project>.vercel.app`
- Separate database: Use Supabase branch databases
- Full feature testing before merging to main
- Automatic cleanup after PR closure

### Production Deployments

```bash
# Trigger via git push to main
git push origin main
# Vercel automatically:
# 1. Builds Next.js app
# 2. Runs checks (lint, type, test)
# 3. Deploys to CDN
# 4. Routes traffic gradually (blue-green)
# 5. Monitors performance metrics
```

### Custom Domains & SSL

```bash
# Via Vercel CLI
vercel alias set <project-url> yourdomain.com

# Via Vercel Dashboard
# Settings → Domains → Add Domain
# Auto-provisions SSL certificate via Let's Encrypt
# DNS records configured automatically
```

### Edge Config for Feature Flags

```js
// lib/edge-config.ts
import { get } from '@vercel/edge-config'

export async function isFeatureEnabled(feature: string) {
  try {
    const flags = await get('features')
    return flags?.[feature] ?? false
  } catch (error) {
    console.error('Edge Config error:', error)
    return false
  }
}

// Usage in API route
import { isFeatureEnabled } from '@/lib/edge-config'

export default async function handler(req, res) {
  const newFeatureEnabled = await isFeatureEnabled('new_checkout')
  if (!newFeatureEnabled) {
    return res.status(404).json({ error: 'Not found' })
  }
  // ... feature code
}
```

## 2. GitHub Actions CI/CD Pipeline

Automate testing, building, and deployment with GitHub Actions workflows.

### Workflow File Structure (.github/workflows/deploy.yml)

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '18'
  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
  TURBO_TEAM: ${{ secrets.TURBO_TEAM }}

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint -- --max-warnings=0

      - name: Type check
        run: npm run type-check

      - name: Run tests
        run: npm run test:ci
        env:
          CI: true

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
          fail_ci_if_error: true

  build:
    needs: lint-and-test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build
        env:
          NEXT_PUBLIC_API_URL: ${{ secrets.NEXT_PUBLIC_API_URL }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Cache build artifacts
        uses: actions/cache@v3
        with:
          path: .next
          key: ${{ runner.os }}-next-${{ github.sha }}

  deploy-preview:
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel (Preview)
        uses: vercel/action@main
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          github-comment: true

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment:
      name: production
      url: https://yourdomain.com

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel (Production)
        uses: vercel/action@main
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          production: true

      - name: Run smoke tests
        run: |
          npm run test:smoke
        env:
          SMOKE_TEST_URL: https://yourdomain.com

      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: "Production deployment completed"
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Database Migrations in CI

```yaml
  migrate-database:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run migrations
        run: npm run migrate:deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Seed production data
        run: npm run seed:prod
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

## 3. The Pipeline: Development to Production

```
lint → type-check → test → build → preview → approve → production
```

### Complete Pipeline Visualization

```
┌─────────────────────────────────────────────────────────────┐
│ Developer pushes code to feature branch                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ GitHub Actions Triggers: lint, type-check, test             │
│ ✓ ESLint, Prettier                                          │
│ ✓ TypeScript compiler check                                 │
│ ✓ Jest unit & integration tests                             │
│ ✓ E2E tests (Playwright/Cypress)                            │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │ Checks Pass?            │
        └────────────┬────────────┘
                     │ Yes
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Build Next.js Application                                   │
│ ✓ next build                                                │
│ ✓ Optimize bundles (code-splitting, tree-shaking)          │
│ ✓ Static generation + ISR setup                             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Deploy to Vercel Preview                                    │
│ ✓ Create unique preview URL                                 │
│ ✓ Spin up Supabase branch database                          │
│ ✓ Post preview link to GitHub PR                            │
│ ✓ Enable stakeholder testing                                │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Code Review & Approval                                      │
│ ✓ Automated checks must pass                                │
│ ✓ Require N peer reviews (typically 2)                      │
│ ✓ Manual testing on preview                                 │
│ ✓ Product sign-off for feature flags                        │
└────────────────────┬────────────────────────────────────────┘
                     │ Approved & Merged
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Production Deployment                                       │
│ ✓ Deploy to Vercel production                               │
│ ✓ Run database migrations                                   │
│ ✓ Gradual traffic rollout (blue-green)                      │
│ ✓ Monitor error rates & performance                         │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ Post-Deploy Verification                                    │
│ ✓ Smoke tests against prod                                  │
│ ✓ Check monitoring dashboards                               │
│ ✓ Notify stakeholders                                       │
│ ✓ Monitor for 30 minutes                                    │
└─────────────────────────────────────────────────────────────┘
```

## 4. Environment Management

### Development Environment
```bash
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=postgresql://user:pass@localhost:5432/dev_db
SUPABASE_URL=http://localhost:54321
SUPABASE_ANON_KEY=eyJhbGc...
LOG_LEVEL=debug
```

### Staging Environment
```bash
# Vercel environment variable (Preview)
NEXT_PUBLIC_API_URL=https://staging-api.yourdomain.com
DATABASE_URL=postgresql://user:pass@db-staging.yourdomain.com/staging
SUPABASE_URL=https://staging-xxx.supabase.co
LOG_LEVEL=info
```

### Production Environment
```bash
# Vercel environment variable (Production)
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
DATABASE_URL=postgresql://readonly:pass@db-prod.yourdomain.com/prod
SUPABASE_URL=https://prod-xxx.supabase.co
LOG_LEVEL=warn
SENTRY_DSN=https://key@sentry.io/project
```

### Environment Variable Pattern

```ts
// lib/env.ts
import { z } from 'zod'

const envSchema = z.object({
  NEXT_PUBLIC_API_URL: z.string().url(),
  DATABASE_URL: z.string().url(),
  SUPABASE_URL: z.string().url(),
  SUPABASE_ANON_KEY: z.string(),
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']),
})

export const env = envSchema.parse({
  NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  DATABASE_URL: process.env.DATABASE_URL,
  SUPABASE_URL: process.env.SUPABASE_URL,
  SUPABASE_ANON_KEY: process.env.SUPABASE_ANON_KEY,
  NODE_ENV: process.env.NODE_ENV,
  LOG_LEVEL: process.env.LOG_LEVEL || 'info',
})
```

## 5. Preview Deployments

### Automatic PR Preview URLs

Every pull request deploys to a unique Vercel preview URL:

```
https://ship-it-pr-42.yourdomain.vercel.app
https://ship-it-main.yourdomain.vercel.app
```

### Supabase Branch Databases

```bash
# Enable Branch functionality in Supabase
# 1. Project Settings → Branches
# 2. Create branch database per PR
# 3. Auto-seed with production schema (via migrations)

# In CI/CD: Create branch before deploy
supabase branches create --branch preview-${GITHUB_HEAD_REF}

# Link to preview environment
SUPABASE_URL=https://preview-${BRANCH}.supabase.co
SUPABASE_ANON_KEY=xxx
```

### Preview Environment Isolation

```ts
// lib/db.ts - Isolate test data in preview
import { createClient } from '@supabase/supabase-js'

const isPreview = process.env.VERCEL_ENV === 'preview'

export const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!,
  {
    auth: {
      persistSession: !isPreview,
    },
  }
)

// Automatically delete preview data after N days
if (isPreview) {
  supabase
    .from('preview_data')
    .delete()
    .lt('created_at', new Date(Date.now() - 7 * 24 * 60 * 60 * 1000))
}
```

## 6. Docker & Container Deployment

### Multi-Stage Dockerfile for Next.js

```dockerfile
# Stage 1: Dependencies
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Runtime
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json

USER nextjs
EXPOSE 3000
ENV PORT=3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

CMD ["npm", "start"]
```

### Docker Compose for Local Development

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/dev
      - SUPABASE_URL=http://supabase:3000
    depends_on:
      - db
      - supabase
    volumes:
      - .:/app
      - /app/node_modules

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  supabase:
    image: supabase/supabase:latest
    ports:
      - "3000:3000"
    environment:
      POSTGRES_PASSWORD: pass
    volumes:
      - supabase_data:/var/lib/postgresql/data

volumes:
  postgres_data:
  supabase_data:
```

## 7. Monitoring & Observability

### Vercel Analytics Integration

```ts
// lib/analytics.ts
import { Analytics } from '@vercel/analytics/react'

export function setupAnalytics() {
  return <Analytics />
}

// pages/_app.tsx
import { setupAnalytics } from '@/lib/analytics'

export default function App({ Component, pageProps }) {
  return (
    <>
      <Component {...pageProps} />
      {setupAnalytics()}
    </>
  )
}
```

### Sentry Error Tracking

```bash
npm install --save-exact @sentry/nextjs
npx @sentry/wizard@latest -i nextjs
```

```js
// next.config.js
const withSentry = require('@sentry/nextjs').withSentryConfig

module.exports = withSentry(
  {
    reactStrictMode: true,
  },
  {
    org: 'your-org',
    project: 'your-project',
    authToken: process.env.SENTRY_AUTH_TOKEN,
  }
)
```

### Health Check Endpoint

```ts
// pages/api/health.ts
import type { NextApiRequest, NextApiResponse } from 'next'
import { supabase } from '@/lib/db'

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    // Check database connectivity
    await supabase.from('_health').select('1').single()

    // Check external API
    const apiRes = await fetch(process.env.NEXT_PUBLIC_API_URL + '/health')
    if (!apiRes.ok) throw new Error('API unhealthy')

    return res.status(200).json({
      status: 'ok',
      timestamp: new Date().toISOString(),
    })
  } catch (error) {
    return res.status(503).json({
      status: 'error',
      error: String(error),
    })
  }
}
```

## 8. Rollback Strategy

### Vercel Instant Rollback

```bash
# In Vercel Dashboard → Deployments
# Click "Rollback" on any previous deployment
# Automatic promotion restores traffic instantly

# Via CLI
vercel rollback
# Prompts for deployment to revert to
```

### Database Rollback Pattern

```ts
// Use migrations with down() functions
// migrations/001_create_users.sql

-- Up
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) NOT NULL
);

-- Down
DROP TABLE users;

// In emergency, run:
// npm run migrate:down -- --target 000_before_incident
```

### Feature Flags as Kill Switches

```ts
// lib/features.ts
import { get } from '@vercel/edge-config'

export const FEATURES = {
  NEW_CHECKOUT: 'new_checkout',
  AI_RECOMMENDATIONS: 'ai_recommendations',
  BETA_UI: 'beta_ui',
}

// Disable instantly without deploy
await fetch('https://edge-config.vercel.sh/patch', {
  method: 'PATCH',
  headers: { Authorization: `Bearer ${TOKEN}` },
  body: JSON.stringify({
    items: [
      { operation: 'update', key: FEATURES.NEW_CHECKOUT, value: false },
    ],
  }),
})
```

## 9. Pre-Deploy Checklist

- [ ] All tests passing (unit, integration, e2e)
- [ ] Type checking passes (no `any` types)
- [ ] Linting passes (no warnings)
- [ ] Bundle size analyzed (check for regressions)
- [ ] Database migrations tested locally
- [ ] Environment variables validated for target environment
- [ ] Feature flags configured correctly
- [ ] Performance metrics reviewed (Lighthouse, Core Web Vitals)
- [ ] Security scan passed (OWASP Top 10)
- [ ] Stakeholders notified of planned deployment
- [ ] Rollback plan documented
- [ ] Monitoring dashboards configured

## 10. Post-Deploy Checklist

- [ ] Smoke tests passing on production
- [ ] Error rates normal (check Sentry/Vercel Analytics)
- [ ] Response times acceptable (check Vercel Analytics)
- [ ] Database queries performing (check monitoring)
- [ ] All feature flags enabled/disabled as intended
- [ ] User-facing flows tested (critical paths)
- [ ] Third-party integrations functional
- [ ] Email/notifications sending correctly
- [ ] CDN cache warmed (key pages)
- [ ] Team notified of successful deployment
- [ ] Monitor for 30 minutes before declaring victory
- [ ] Document any issues discovered

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
