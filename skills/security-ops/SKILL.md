---
name: security-ops
description: >
  Web application security for Next.js and Supabase stacks. Master OWASP Top 10 vulnerabilities including injection attacks, broken authentication, XSS, CSRF, and sensitive data exposure. Implement production-grade security headers, Content Security Policy, Row-Level Security, input validation, and secrets management. Harden authentication with JWT best practices, MFA, PKCE flow, and session management. Prevent SQL injection, XXE attacks, and insecure deserialization. API security, rate limiting, webhook verification, and security audit checklists included. CSP headers, HSTS, sanitization patterns, and NEXT_PUBLIC_ environment variable rules. Built-in Server Actions CSRF protection, DOMPurify integration, parameterized queries, and OAuth state parameter validation. Complete vulnerability remediation patterns for the modern web stack.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "OWASP Top 10 2025, Next.js security, Supabase security, web security best practices"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## OWASP Top 10: Mitigation Patterns

### 1. Injection Attacks

**SQL Injection Prevention:**
```typescript
// ❌ VULNERABLE
const user = await db.query(`SELECT * FROM users WHERE id = ${userId}`);

// ✅ SAFE - Parameterized queries
const { data, error } = await supabase
  .from('users')
  .select('*')
  .eq('id', userId);
```

**NoSQL Injection Prevention:**
```typescript
// Always validate and sanitize input
import { z } from 'zod';

const userIdSchema = z.string().uuid();
const userId = userIdSchema.parse(request.body.id);
```

### 2. Broken Authentication

**JWT Security:**
```typescript
// app/api/auth/verify.ts
export async function POST(req: Request) {
  const token = req.headers.get('authorization')?.split(' ')[1];

  if (!token) return Response.json({ error: 'Unauthorized' }, { status: 401 });

  const { data, error } = await supabase.auth.getUser(token);
  if (error) return Response.json({ error: 'Invalid token' }, { status: 401 });

  return Response.json({ user: data.user });
}
```

**Session Management Best Practices:**
```typescript
// Rotate session tokens regularly
// Set secure cookie flags
response.headers.set('Set-Cookie',
  `sessionToken=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`
);

// Implement timeout and refresh logic
const sessionExpiry = Date.now() + (60 * 60 * 1000); // 1 hour
```

**MFA Implementation:**
```typescript
// Enable MFA in Supabase
const { data, error } = await supabase.auth.mfa.enroll({
  issuer: 'MyApp',
  friendlyName: 'authenticator'
});

// Verify TOTP
const verified = await supabase.auth.mfa.verify({
  factorId: factor.id,
  code: userTotpCode
});
```

**OAuth PKCE Flow:**
```typescript
// Generate code_challenge for PKCE
const codeVerifier = crypto.getRandomValues(new Uint8Array(32));
const base64url = (buf) => btoa(String.fromCharCode(...buf))
  .replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');
const codeChallenge = base64url(new Uint8Array(
  await crypto.subtle.digest('SHA-256', new TextEncoder().encode(codeVerifier))
));

const url = new URL('https://auth.example.com/oauth/authorize');
url.searchParams.set('code_challenge', codeChallenge);
url.searchParams.set('code_challenge_method', 'S256');
```

### 3. Sensitive Data Exposure

**Environment Variables:**
```bash
# .env.local (NEVER commit)
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx

# Server-only secrets
DATABASE_PASSWORD=xxx
JWT_SECRET=xxx
STRIPE_SECRET_KEY=xxx
```

**Encryption at Rest:**
```typescript
// Use Supabase encryption for sensitive fields
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(url, key, {
  auth: {
    persistSession: true,
    autoRefreshToken: true
  }
});
```

### 4. XML External Entity (XXE)

```typescript
// Disable XML external entities parsing
import xml2js from 'xml2js';

const parser = new xml2js.Parser({
  strict: true,
  resolvexmlEntities: false, // ✅ Disable XXE
  doctype: false,
  cdata: true
});
```

### 5. Broken Access Control

**Row-Level Security (RLS) - The Backbone:**
```sql
-- Enable RLS on all tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own data
CREATE POLICY "Users can see own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- Policy: Users can only update their own posts
CREATE POLICY "Users can update own posts"
  ON posts FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- Policy: Users can only delete their own data
CREATE POLICY "Users can delete own data"
  ON users FOR DELETE
  USING (auth.uid() = id);
```

**Middleware Route Protection:**
```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')?.value;

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*']
};
```

**Role-Based Access Control:**
```typescript
// lib/auth.ts
export async function checkRole(userId: string, requiredRole: string) {
  const { data: user } = await supabase
    .from('users')
    .select('role')
    .eq('id', userId)
    .single();

  const roleHierarchy = { admin: 3, moderator: 2, user: 1 };
  return roleHierarchy[user?.role] >= roleHierarchy[requiredRole];
}
```

### 6. Security Misconfiguration

**Next.js Security Headers:**
```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  headers: async () => [
    {
      source: '/:path*',
      headers: [
        {
          key: 'Content-Security-Policy',
          value: "default-src 'self'; script-src 'self' 'unsafe-inline'; img-src 'self' data: https:; style-src 'self' 'unsafe-inline';"
        },
        {
          key: 'Strict-Transport-Security',
          value: 'max-age=31536000; includeSubDomains; preload'
        },
        {
          key: 'X-Frame-Options',
          value: 'DENY'
        },
        {
          key: 'X-Content-Type-Options',
          value: 'nosniff'
        },
        {
          key: 'Referrer-Policy',
          value: 'strict-origin-when-cross-origin'
        },
        {
          key: 'Permissions-Policy',
          value: 'geolocation=(), microphone=(), camera=()'
        },
        {
          key: 'X-XSS-Protection',
          value: '1; mode=block'
        }
      ]
    }
  ]
};

module.exports = nextConfig;
```

### 7. Cross-Site Scripting (XSS)

**React Built-in Protection:**
```typescript
// ✅ React automatically escapes text content
function UserProfile({ user }) {
  return <div>{user.name}</div>; // Safe, even with "<script>" in name
}

// ❌ DANGEROUS - Never use dangerouslySetInnerHTML
function BadComponent({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// ✅ SAFE - Sanitize before render
import DOMPurify from 'dompurify';

function SafeHTML({ html }) {
  const clean = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

**Content Security Policy Headers:**
```typescript
// Block inline scripts, only allow HTTPS resources
const csp = "default-src 'self'; script-src 'self' 'nonce-${nonce}'; img-src 'self' data: https:;";
```

### 8. Insecure Deserialization

```typescript
// ❌ VULNERABLE - eval() is dangerous
const data = eval(`(${userInput})`);

// ✅ SAFE - Use JSON parsing with validation
import { z } from 'zod';

const userDataSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  age: z.number().min(0).max(150)
});

const data = userDataSchema.parse(JSON.parse(userInput));
```

### 9. Components with Known Vulnerabilities

**Dependency Scanning:**
```bash
# Regular audits
npm audit

# Fix vulnerabilities
npm audit fix

# Use Snyk or Dependabot for continuous monitoring
```

### 10. Insufficient Logging & Monitoring

```typescript
// app/api/suspicious-action/route.ts
import { log } from 'lib/logger';

export async function POST(req: Request) {
  const user = await getCurrentUser();
  const action = req.body.action;

  try {
    // Execute action
    log.info(`User ${user.id} performed action: ${action}`, {
      timestamp: new Date(),
      userId: user.id,
      action: action,
      ip: req.headers.get('x-forwarded-for')
    });
  } catch (error) {
    log.error(`Security event: Failed action by ${user.id}`, {
      error: error.message,
      userId: user.id,
      severity: 'high'
    });
    // Alert security team
  }
}
```

## Input Validation with Zod

**Server and Client Validation:**
```typescript
// lib/schemas.ts
import { z } from 'zod';

export const createPostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(10).max(5000),
  tags: z.array(z.string()).max(5),
  publishedAt: z.date().optional()
});

// app/actions/posts.ts - Server Action
'use server';

export async function createPost(formData: FormData) {
  const raw = Object.fromEntries(formData);
  const validated = createPostSchema.parse(raw);

  const { data, error } = await supabase
    .from('posts')
    .insert([{ ...validated, user_id: user.id }]);

  return { success: !error, error: error?.message };
}

// components/CreatePostForm.tsx - Client
'use client';

export function CreatePostForm() {
  const [errors, setErrors] = useState({});

  async function handleSubmit(formData: FormData) {
    try {
      const result = await createPost(formData);
      if (!result.success) setErrors({ submit: result.error });
    } catch (error) {
      setErrors({ submit: 'Validation failed' });
    }
  }

  return <form action={handleSubmit}>{/* ... */}</form>;
}
```

## CSRF Protection

**Server Actions (Built-in Protection):**
```typescript
// ✅ Server Actions automatically validate CSRF tokens
'use server';

export async function deleteAccount() {
  const user = await getCurrentUser();
  // CSRF validation happens automatically
  await supabase.auth.signOut();
}

// ✅ API Routes - Add CSRF token validation
import { csrf } from 'lib/csrf';

export async function POST(req: Request) {
  await csrf.validate(req);
  // Process request
}
```

**SameSite Cookies:**
```typescript
response.headers.set('Set-Cookie',
  `token=${value}; SameSite=Strict; Secure; HttpOnly`
);
```

## Secrets Management

**Secure Pattern:**
```typescript
// lib/secrets.ts
function requireServerSecret(key: string): string {
  const value = process.env[key];
  if (!value) {
    throw new Error(`Missing required secret: ${key}`);
  }
  return value;
}

// Usage in Server Actions only
const dbPassword = requireServerSecret('DATABASE_PASSWORD');
const jwtSecret = requireServerSecret('JWT_SECRET');

// ❌ Never use in client
export const apiKey = process.env.STRIPE_SECRET_KEY; // Wrong!
```

## API Security

**Rate Limiting:**
```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h')
});

export async function POST(req: Request) {
  const ip = req.headers.get('x-forwarded-for');
  const { success } = await ratelimit.limit(ip);

  if (!success) {
    return Response.json({ error: 'Rate limited' }, { status: 429 });
  }

  // Process request
}
```

**Webhook Signature Verification:**
```typescript
import crypto from 'crypto';

export function verifyWebhookSignature(
  payload: string,
  signature: string,
  secret: string
): boolean {
  const hash = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(hash),
    Buffer.from(signature)
  );
}
```

## Security Audit Checklist

- [ ] All RLS policies enabled and tested
- [ ] HTTPS enforced (HSTS headers set)
- [ ] CSP headers configured and strict
- [ ] No secrets in .env (only in Vercel/Railway secrets)
- [ ] All user inputs validated with Zod
- [ ] File uploads restricted by type and size
- [ ] Rate limiting on API routes
- [ ] CSRF protection on state-changing actions
- [ ] Password policies enforced (min 12 chars)
- [ ] MFA available and recommended
- [ ] Dependency vulnerabilities scanned weekly
- [ ] Error messages don't leak sensitive info
- [ ] Logging configured for suspicious activity
- [ ] CORS configured restrictively
- [ ] API keys rotated monthly
- [ ] Database backups encrypted and tested
- [ ] Authentication sessions timeout after 1 hour
- [ ] Refresh tokens rotate on use
- [ ] No console logs of sensitive data
- [ ] Security headers tested with Mozilla Observatory

<sub>Open RX by CutTheChexx — The Prescription.</sub>
