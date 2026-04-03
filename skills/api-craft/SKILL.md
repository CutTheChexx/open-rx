---
name: api-craft
description: >
  Production-grade API design and backend patterns for modern web applications.
  Master Supabase Edge Functions, Next.js API routes, REST design principles,
  input validation with Zod, webhook handling, cron jobs, authentication flows,
  and enterprise-grade error handling. Build robust, scalable APIs.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Supabase docs, Next.js API Routes, REST API best practices, OWASP security guidelines"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## api-craft: Backend Mastery

The `api-craft` skill is your comprehensive guide to building production-grade APIs. Whether you're deploying edge functions, creating Next.js route handlers, or integrating webhooks, this skill covers the full spectrum of modern backend development.

---

## 1. Supabase Edge Functions

### Getting Started

Edge Functions run on Deno and execute at the edge—globally distributed, low latency.

```bash
supabase functions new my-function
```

A basic function structure:

```typescript
// supabase/functions/my-function/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS",
  "Access-Control-Allow-Headers": "authorization,x-client-info,content-type",
};

serve(async (req) => {
  // Handle CORS preflight
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    const { name } = await req.json();

    return new Response(
      JSON.stringify({ message: `Hello, ${name}!` }),
      {
        headers: { ...corsHeaders, "Content-Type": "application/json" },
        status: 200,
      }
    );
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      headers: { ...corsHeaders, "Content-Type": "application/json" },
      status: 400,
    });
  }
});
```

### Environment Variables

Store secrets in `.env.local`:

```bash
OPENAI_API_KEY=sk-...
DATABASE_PASSWORD=...
```

Access them:

```typescript
const apiKey = Deno.env.get("OPENAI_API_KEY");
```

### Calling Supabase from Edge Functions

```typescript
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.0.0";

const supabaseUrl = Deno.env.get("SUPABASE_URL");
const supabaseAnonKey = Deno.env.get("SUPABASE_ANON_KEY");

const supabase = createClient(supabaseUrl, supabaseAnonKey);

// Query from edge
const { data, error } = await supabase
  .from("users")
  .select("*")
  .eq("id", userId);
```

### Deployment

```bash
supabase functions deploy my-function
supabase functions deploy my-function --no-verify-jwt
```

Check logs:

```bash
supabase functions fetch-logs my-function
```

---

## 2. Next.js API Routes (App Router)

### Basic Route Handler

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const id = searchParams.get("id");

    if (!id) {
      return NextResponse.json({ error: "Missing id" }, { status: 400 });
    }

    // Fetch from database
    const data = await db.users.findUnique({ where: { id } });

    return NextResponse.json(data);
  } catch (error) {
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    // Validate with Zod
    const validated = userSchema.parse(body);
    // Create in database
    const user = await db.users.create({ data: validated });
    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 400 });
  }
}
```

### Dynamic Routes

```typescript
// app/api/users/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.users.findUnique({ where: { id: params.id } });
  if (!user) {
    return NextResponse.json({ error: "Not found" }, { status: 404 });
  }
  return NextResponse.json(user);
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const updated = await db.users.update({
    where: { id: params.id },
    data: body,
  });
  return NextResponse.json(updated);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await db.users.delete({ where: { id: params.id } });
  return new NextResponse(null, { status: 204 });
}
```

### Middleware

```typescript
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

export function middleware(request: NextRequest) {
  // Add custom headers
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set("x-pathname", request.nextUrl.pathname);

  return NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });
}

export const config = {
  matcher: ["/api/:path*"],
};
```

### Streaming Responses

```typescript
// app/api/stream/route.ts
export async function GET() {
  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 100; i++) {
        const data = JSON.stringify({ count: i });
        controller.enqueue(data + "\n");
        await new Promise((resolve) => setTimeout(resolve, 100));
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "application/ndjson",
      "Transfer-Encoding": "chunked",
    },
  });
}
```

---

## 3. Server Actions vs API Routes

### When to Use Server Actions

- Form submissions from server components
- Simple mutations with no external consumers
- Direct database operations with RLS policies
- Client → Server tightly coupled

```typescript
// app/actions/create-user.ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";

const schema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

export async function createUser(formData: FormData) {
  const rawFormData = {
    name: formData.get("name"),
    email: formData.get("email"),
  };

  const validated = schema.parse(rawFormData);
  const user = await db.users.create({ data: validated });

  revalidatePath("/users");
  return user;
}
```

```tsx
// components/user-form.tsx
"use client";

import { createUser } from "@/app/actions/create-user";

export function UserForm() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

### When to Use API Routes

- Webhooks from external services (Stripe, GitHub)
- Public endpoints with CORS
- Service-to-service communication
- Third-party integrations with API keys
- Complex query strings and pagination
- Rate limiting per API key

---

## 4. REST API Design

### URL Patterns

```
GET    /api/users                          # List all users
GET    /api/users/[id]                     # Get single user
POST   /api/users                          # Create user
PATCH  /api/users/[id]                     # Update user
DELETE /api/users/[id]                     # Delete user
GET    /api/users/[id]/posts               # List user's posts
POST   /api/users/[id]/posts               # Create post for user
```

### HTTP Status Codes

```
200 OK                  - Successful GET, PATCH
201 Created            - Successful POST
204 No Content         - Successful DELETE
400 Bad Request        - Invalid input
401 Unauthorized       - Missing/invalid auth
403 Forbidden          - Authenticated but denied
404 Not Found          - Resource doesn't exist
409 Conflict           - Duplicate or conflict
422 Unprocessable      - Validation error
429 Too Many Requests  - Rate limited
500 Server Error       - Unhandled exception
503 Service Unavailable - Temporary issue
```

### Pagination with Link Headers

```typescript
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get("page") || "1");
  const limit = parseInt(searchParams.get("limit") || "20");

  const total = await db.users.count();
  const users = await db.users.findMany({
    skip: (page - 1) * limit,
    take: limit,
  });

  const lastPage = Math.ceil(total / limit);

  const links: string[] = [];
  if (page > 1) links.push(`<${baseUrl}?page=1>; rel="first"`);
  if (page > 1) links.push(`<${baseUrl}?page=${page - 1}>; rel="prev"`);
  if (page < lastPage) links.push(`<${baseUrl}?page=${page + 1}>; rel="next"`);
  if (page < lastPage) links.push(`<${baseUrl}?page=${lastPage}>; rel="last"`);

  return NextResponse.json(
    { data: users, total, page, lastPage },
    {
      headers: {
        "Link": links.join(", "),
        "X-Total-Count": total.toString(),
      },
    }
  );
}
```

### Cursor-Based Pagination

```typescript
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const cursor = searchParams.get("cursor");
  const limit = 20;

  const items = await db.posts.findMany({
    take: limit + 1,
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: "desc" },
  });

  const hasMore = items.length > limit;
  const data = hasMore ? items.slice(0, -1) : items;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  return NextResponse.json({
    data,
    nextCursor,
    hasMore,
  });
}
```

### Filtering and Sorting

```typescript
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);

  const where: any = {};

  // Filter by status
  if (searchParams.has("status")) {
    where.status = searchParams.get("status");
  }

  // Filter by date range
  if (searchParams.has("from") && searchParams.has("to")) {
    where.createdAt = {
      gte: new Date(searchParams.get("from")),
      lte: new Date(searchParams.get("to")),
    };
  }

  // Sort
  const sortBy = searchParams.get("sortBy") || "createdAt";
  const sortOrder = searchParams.get("order") === "asc" ? "asc" : "desc";

  const items = await db.items.findMany({
    where,
    orderBy: { [sortBy]: sortOrder },
    take: 50,
  });

  return NextResponse.json(items);
}
```

### Error Response Format

```typescript
interface ApiError {
  error: string;
  code: string;
  details?: Record<string, unknown>;
  timestamp: string;
}

// Usage
return NextResponse.json(
  {
    error: "Validation failed",
    code: "VALIDATION_ERROR",
    details: {
      email: "Invalid email format",
      age: "Must be 18 or older",
    },
    timestamp: new Date().toISOString(),
  },
  { status: 422 }
);
```

### API Versioning

```typescript
// Option 1: URL path versioning
// /api/v1/users, /api/v2/users

// Option 2: Header versioning
const version = request.headers.get("api-version") || "1";

if (version === "2") {
  // New response format
  return NextResponse.json({ data: users, meta: { total } });
} else {
  // Legacy format
  return NextResponse.json(users);
}
```

---

## 5. Input Validation with Zod

### Basic Schemas

```typescript
import { z } from "zod";

const userSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email"),
  age: z.number().int().gte(0).lte(150).optional(),
  role: z.enum(["admin", "user", "moderator"]),
  tags: z.array(z.string()).max(5),
  metadata: z.record(z.string(), z.any()).optional(),
});

type User = z.infer<typeof userSchema>;
```

### Endpoint with Validation

```typescript
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validated = userSchema.parse(body);

    const user = await db.users.create({ data: validated });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          error: "Validation failed",
          issues: error.issues.map((issue) => ({
            path: issue.path.join("."),
            message: issue.message,
          })),
        },
        { status: 422 }
      );
    }
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

### Coercion and Transformation

```typescript
const querySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().positive().max(100).default(20),
  search: z.string().optional().transform((s) => s?.trim()),
  sortBy: z.enum(["name", "email", "createdAt"]).default("createdAt"),
});

// Usage with query params
const params = querySchema.parse({
  page: searchParams.get("page"),
  limit: searchParams.get("limit"),
  search: searchParams.get("search"),
  sortBy: searchParams.get("sortBy"),
});
```

---

## 6. Authentication

### JWT Verification

```typescript
import { jwtVerify } from "jose";

const secret = new TextEncoder().encode(process.env.JWT_SECRET!);

export async function verifyAuth(
  request: NextRequest
): Promise<{ userId: string } | null> {
  const authHeader = request.headers.get("authorization");

  if (!authHeader?.startsWith("Bearer ")) {
    return null;
  }

  const token = authHeader.slice(7);

  try {
    const verified = await jwtVerify(token, secret);
    return { userId: verified.payload.sub as string };
  } catch {
    return null;
  }
}

// In route handler
export async function GET(request: NextRequest) {
  const auth = await verifyAuth(request);

  if (!auth) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Continue with authenticated request
  return NextResponse.json({ userId: auth.userId });
}
```

### API Key Authentication

```typescript
export async function verifyApiKey(
  request: NextRequest
): Promise<{ clientId: string; tier: string } | null> {
  const apiKey = request.headers.get("x-api-key");

  if (!apiKey) {
    return null;
  }

  const client = await db.apiClients.findUnique({
    where: { key: apiKey },
  });

  if (!client || !client.active) {
    return null;
  }

  return { clientId: client.id, tier: client.tier };
}

export async function POST(request: NextRequest) {
  const auth = await verifyApiKey(request);

  if (!auth) {
    return NextResponse.json({ error: "Invalid API key" }, { status: 401 });
  }

  // Use auth.tier for rate limiting or feature gates
  return NextResponse.json({ success: true });
}
```

### Rate Limiting

```typescript
// Middleware or within handler
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, "1 h"),
});

export async function GET(request: NextRequest) {
  const auth = await verifyApiKey(request);

  if (auth) {
    // Rate limit per API key
    const { success } = await ratelimit.limit(`api_key_${auth.clientId}`);

    if (!success) {
      return NextResponse.json(
        { error: "Rate limit exceeded" },
        { status: 429 }
      );
    }
  }

  return NextResponse.json({ success: true });
}
```

---

## 7. Webhook Patterns

### Receiving & Validating Webhooks

```typescript
import { Webhook } from "svix";

const SECRET = process.env.WEBHOOK_SECRET!;

export async function POST(request: NextRequest) {
  const body = await request.text();
  const headers = {
    "svix-id": request.headers.get("svix-id")!,
    "svix-timestamp": request.headers.get("svix-timestamp")!,
    "svix-signature": request.headers.get("svix-signature")!,
  };

  let event;

  try {
    const wh = new Webhook(SECRET);
    event = wh.verify(body, headers) as any;
  } catch (error) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Process event
  switch (event.type) {
    case "user.created":
      await handleUserCreated(event.data);
      break;
    case "user.updated":
      await handleUserUpdated(event.data);
      break;
  }

  return NextResponse.json({ received: true });
}
```

### Stripe Webhook

```typescript
import Stripe from "stripe";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const SECRET = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(request: NextRequest) {
  const body = await request.text();
  const sig = request.headers.get("stripe-signature")!;

  let event;

  try {
    event = stripe.webhooks.constructEvent(body, sig, SECRET);
  } catch (error) {
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }

  switch (event.type) {
    case "payment_intent.succeeded":
      await handlePaymentSucceeded(event.data.object);
      break;
    case "charge.dispute.created":
      await handleDispute(event.data.object);
      break;
  }

  return NextResponse.json({ received: true });
}
```

### Idempotent Webhook Processing

```typescript
export async function POST(request: NextRequest) {
  const eventId = request.headers.get("x-webhook-id")!;

  // Check if already processed
  const existing = await db.webhookEvents.findUnique({
    where: { eventId },
  });

  if (existing) {
    return NextResponse.json({ received: true });
  }

  try {
    const body = await request.json();

    // Process webhook
    await handleWebhook(body);

    // Record processed event
    await db.webhookEvents.create({
      data: {
        eventId,
        type: body.type,
        processedAt: new Date(),
      },
    });

    return NextResponse.json({ received: true });
  } catch (error) {
    return NextResponse.json(
      { error: "Processing failed" },
      { status: 500 }
    );
  }
}
```

### Webhook Retry Queue

```typescript
export async function POST(request: NextRequest) {
  const body = await request.json();

  try {
    // Attempt processing
    await handleWebhook(body);
    return NextResponse.json({ received: true });
  } catch (error) {
    // Queue for retry
    const maxRetries = 5;
    const retryDelay = 300; // 5 minutes

    await db.webhookQueue.create({
      data: {
        payload: body,
        retries: 0,
        maxRetries,
        nextRetryAt: new Date(Date.now() + retryDelay * 1000),
        error: error.message,
      },
    });

    return NextResponse.json(
      { queued: true, message: "Will retry" },
      { status: 202 }
    );
  }
}
```

---

## 8. Cron Jobs

### pg_cron in Supabase

```sql
-- Run every day at 2 AM
SELECT cron.schedule(
  'cleanup-sessions',
  '0 2 * * *',
  $$DELETE FROM sessions WHERE expires_at < NOW()$$
);

-- Run every hour
SELECT cron.schedule(
  'sync-cache',
  '0 * * * *',
  $$SELECT update_cache();$$
);

-- Run every Monday at 9 AM
SELECT cron.schedule(
  'weekly-digest',
  '0 9 * * 1',
  $$SELECT send_weekly_digest();$$
);
```

### Edge Function as Cron

```typescript
// supabase/functions/scheduled-cleanup/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";

serve(async (req) => {
  // Verify it's from Supabase (internal)
  const authHeader = req.headers.get("authorization");

  if (!authHeader?.startsWith("Bearer ")) {
    return new Response("Unauthorized", { status: 401 });
  }

  try {
    // Import Supabase client
    const supabase = createClient(
      Deno.env.get("SUPABASE_URL"),
      Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")
    );

    // Cleanup expired sessions
    await supabase
      .from("sessions")
      .delete()
      .lt("expires_at", new Date().toISOString());

    // Send digest emails
    const users = await supabase.from("users").select("*");

    for (const user of users.data || []) {
      await sendDigest(user.email);
    }

    return new Response(
      JSON.stringify({
        success: true,
        message: "Cron job completed",
      }),
      { status: 200, headers: { "Content-Type": "application/json" } }
    );
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { "Content-Type": "application/json" },
    });
  }
});
```

### Trigger from External Service

```bash
# Use external service to hit endpoint daily
curl -X POST https://your-app.com/api/cron/cleanup \
  -H "Authorization: Bearer $(cat /path/to/cron-secret.txt)"
```

---

## 9. Error Handling

### Structured Logging

```typescript
interface LogEntry {
  timestamp: string;
  level: "info" | "warn" | "error";
  endpoint: string;
  userId?: string;
  message: string;
  data?: any;
  duration: number;
}

async function logRequest(
  endpoint: string,
  startTime: number,
  level: "info" | "warn" | "error",
  message: string,
  data?: any,
  userId?: string
) {
  const entry: LogEntry = {
    timestamp: new Date().toISOString(),
    level,
    endpoint,
    userId,
    message,
    data,
    duration: Date.now() - startTime,
  };

  console.log(JSON.stringify(entry));

  // Send to external logging service
  if (level === "error") {
    await fetch("https://logs.example.com/api/logs", {
      method: "POST",
      body: JSON.stringify(entry),
      headers: { "Authorization": `Bearer ${process.env.LOG_TOKEN}` },
    });
  }
}
```

### Retry with Exponential Backoff

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      const delay = baseDelay * Math.pow(2, i);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
  throw new Error("Retries exhausted");
}

// Usage
const data = await withRetry(() => fetchFromExternalApi());
```

### Circuit Breaker Pattern

```typescript
class CircuitBreaker {
  private failures = 0;
  private lastFailureTime = 0;
  private state: "closed" | "open" | "half-open" = "closed";

  constructor(
    private threshold = 5,
    private timeout = 60000
  ) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === "open") {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = "half-open";
      } else {
        throw new Error("Circuit breaker is open");
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = "closed";
  }

  private onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();
    if (this.failures >= this.threshold) {
      this.state = "open";
    }
  }
}

// Usage
const breaker = new CircuitBreaker();

export async function GET(request: NextRequest) {
  try {
    const data = await breaker.execute(() => fetchExternalApi());
    return NextResponse.json(data);
  } catch (error) {
    return NextResponse.json(
      { error: "Service temporarily unavailable" },
      { status: 503 }
    );
  }
}
```

---

## 10. API Security

### CORS Configuration

```typescript
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

const allowedOrigins = (process.env.ALLOWED_ORIGINS || "").split(",");

export function middleware(request: NextRequest) {
  const origin = request.headers.get("origin");

  if (!allowedOrigins.includes(origin || "")) {
    return new NextResponse(null, { status: 403 });
  }

  const response = NextResponse.next();

  response.headers.set("Access-Control-Allow-Origin", origin || "");
  response.headers.set(
    "Access-Control-Allow-Methods",
    "GET,POST,PATCH,DELETE,OPTIONS"
  );
  response.headers.set(
    "Access-Control-Allow-Headers",
    "authorization,content-type"
  );
  response.headers.set("Access-Control-Allow-Credentials", "true");
  response.headers.set("Access-Control-Max-Age", "86400");

  if (request.method === "OPTIONS") {
    return response;
  }

  return response;
}

export const config = {
  matcher: ["/api/:path*"],
};
```

### Request Size Limits

```typescript
export async function POST(request: NextRequest) {
  const contentLength = request.headers.get("content-length");

  if (contentLength && parseInt(contentLength) > 10 * 1024 * 1024) {
    return NextResponse.json(
      { error: "Payload too large" },
      { status: 413 }
    );
  }

  try {
    const body = await request.json();
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json(
      { error: "Invalid JSON" },
      { status: 400 }
    );
  }
}
```

### Input Sanitization

```typescript
import { z } from "zod";
import DOMPurify from "isomorphic-dompurify";

const schema = z.object({
  content: z.string().transform((val) => DOMPurify.sanitize(val)),
  email: z.string().email().toLowerCase().trim(),
});

export async function POST(request: NextRequest) {
  const body = await request.json();

  try {
    const sanitized = schema.parse(body);
    // Sanitized input is safe
    await db.posts.create({ data: sanitized });
    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 400 });
  }
}
```

### SQL Injection Prevention (Using Prisma)

```typescript
// Always use parameterized queries
const user = await db.users.findUnique({
  where: { id: userId }, // Parameterized
});

// Never use string concatenation
// WRONG: const user = db.$queryRaw`SELECT * FROM users WHERE id = ${userId}`
```

---

## Deployment Checklist

- Environment variables set (.env.production)
- Rate limiting configured
- CORS origins whitelisted
- Auth tokens rotated
- Logging configured
- Error monitoring (Sentry, Datadog)
- Database connection pooling
- API documentation generated
- Load testing completed
- Security headers configured
- Webhook signatures verified
- Database backups scheduled

<sub>Open RX by CutTheChexx — The Prescription.</sub>
