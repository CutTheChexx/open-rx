# Endpoint Recipes

Production-ready code templates for common API patterns.

---

## Recipe 1: Supabase Edge Function Template

Full template with CORS, auth, error handling, and logging.

```typescript
// supabase/functions/api-handler/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.0.0";

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET,POST,PUT,DELETE,OPTIONS",
  "Access-Control-Allow-Headers": "authorization,x-client-info,content-type",
};

interface RequestContext {
  supabase: ReturnType<typeof createClient>;
  auth: { userId: string } | null;
  body: any;
  headers: Headers;
}

async function verifyAuth(headers: Headers): Promise<{ userId: string } | null> {
  const authHeader = headers.get("authorization");

  if (!authHeader?.startsWith("Bearer ")) {
    return null;
  }

  const token = authHeader.slice(7);

  try {
    const supabase = createClient(
      Deno.env.get("SUPABASE_URL")!,
      Deno.env.get("SUPABASE_ANON_KEY")!
    );

    const { data: { user }, error } = await supabase.auth.getUser(token);

    if (error || !user) {
      return null;
    }

    return { userId: user.id };
  } catch {
    return null;
  }
}

async function handleRequest(req: Request): Promise<Response> {
  // Handle CORS preflight
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    const supabase = createClient(
      Deno.env.get("SUPABASE_URL")!,
      Deno.env.get("SUPABASE_ANON_KEY")!
    );

    const auth = await verifyAuth(req.headers);
    const body = req.method !== "GET" ? await req.json() : null;

    const context: RequestContext = {
      supabase,
      auth,
      body,
      headers: req.headers,
    };

    // Route by method
    switch (req.method) {
      case "GET":
        return await handleGet(context);
      case "POST":
        return await handlePost(context);
      case "PUT":
        return await handlePut(context);
      case "DELETE":
        return await handleDelete(context);
      default:
        return errorResponse("Method not allowed", 405);
    }
  } catch (error) {
    console.error("Unhandled error:", error);
    return errorResponse("Internal server error", 500);
  }
}

async function handleGet(ctx: RequestContext): Promise<Response> {
  if (!ctx.auth) {
    return errorResponse("Unauthorized", 401);
  }

  const { data, error } = await ctx.supabase
    .from("items")
    .select("*")
    .eq("user_id", ctx.auth.userId);

  if (error) {
    return errorResponse(error.message, 400);
  }

  return successResponse(data);
}

async function handlePost(ctx: RequestContext): Promise<Response> {
  if (!ctx.auth) {
    return errorResponse("Unauthorized", 401);
  }

  if (!ctx.body?.name) {
    return errorResponse("Missing required field: name", 422);
  }

  const { data, error } = await ctx.supabase
    .from("items")
    .insert({
      name: ctx.body.name,
      user_id: ctx.auth.userId,
      created_at: new Date().toISOString(),
    })
    .select();

  if (error) {
    return errorResponse(error.message, 400);
  }

  return successResponse(data[0], 201);
}

async function handlePut(ctx: RequestContext): Promise<Response> {
  if (!ctx.auth) {
    return errorResponse("Unauthorized", 401);
  }

  const { data, error } = await ctx.supabase
    .from("items")
    .update(ctx.body)
    .eq("user_id", ctx.auth.userId)
    .select();

  if (error) {
    return errorResponse(error.message, 400);
  }

  return successResponse(data);
}

async function handleDelete(ctx: RequestContext): Promise<Response> {
  if (!ctx.auth) {
    return errorResponse("Unauthorized", 401);
  }

  const { error } = await ctx.supabase
    .from("items")
    .delete()
    .eq("user_id", ctx.auth.userId);

  if (error) {
    return errorResponse(error.message, 400);
  }

  return new Response(null, {
    status: 204,
    headers: corsHeaders,
  });
}

function successResponse(data: any, status = 200): Response {
  return new Response(
    JSON.stringify({
      success: true,
      data,
      timestamp: new Date().toISOString(),
    }),
    {
      status,
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    }
  );
}

function errorResponse(message: string, status: number): Response {
  return new Response(
    JSON.stringify({
      success: false,
      error: message,
      timestamp: new Date().toISOString(),
    }),
    {
      status,
      headers: { ...corsHeaders, "Content-Type": "application/json" },
    }
  );
}

serve(handleRequest);
```

---

## Recipe 2: Next.js Route Handler Template

Complete CRUD endpoint with validation, auth, and error handling.

```typescript
// app/api/items/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { verifyAuth } from "@/lib/auth";
import { db } from "@/lib/db";

const itemSchema = z.object({
  name: z.string().min(1, "Name required").max(200),
  description: z.string().max(1000).optional(),
  completed: z.boolean().default(false),
});

type Item = z.infer<typeof itemSchema>;

export async function GET(request: NextRequest) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get("page") || "1");
    const limit = parseInt(searchParams.get("limit") || "20");
    const completed = searchParams.get("completed");

    const where: any = { userId: auth.userId };

    if (completed !== null) {
      where.completed = completed === "true";
    }

    const [items, total] = await Promise.all([
      db.items.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { createdAt: "desc" },
      }),
      db.items.count({ where }),
    ]);

    const lastPage = Math.ceil(total / limit);

    return NextResponse.json({
      data: items,
      pagination: {
        page,
        limit,
        total,
        lastPage,
      },
    });
  } catch (error) {
    console.error("[GET /api/items]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const validated = itemSchema.parse(body);

    const item = await db.items.create({
      data: {
        ...validated,
        userId: auth.userId,
      },
    });

    return NextResponse.json(item, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          error: "Validation failed",
          issues: error.issues,
        },
        { status: 422 }
      );
    }

    console.error("[POST /api/items]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

```typescript
// app/api/items/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { verifyAuth } from "@/lib/auth";
import { db } from "@/lib/db";

const updateSchema = z.object({
  name: z.string().min(1).max(200).optional(),
  description: z.string().max(1000).optional(),
  completed: z.boolean().optional(),
});

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const item = await db.items.findFirst({
      where: {
        id: params.id,
        userId: auth.userId,
      },
    });

    if (!item) {
      return NextResponse.json({ error: "Not found" }, { status: 404 });
    }

    return NextResponse.json(item);
  } catch (error) {
    console.error("[GET /api/items/[id]]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    // Verify ownership
    const item = await db.items.findFirst({
      where: {
        id: params.id,
        userId: auth.userId,
      },
    });

    if (!item) {
      return NextResponse.json({ error: "Not found" }, { status: 404 });
    }

    const body = await request.json();
    const validated = updateSchema.parse(body);

    const updated = await db.items.update({
      where: { id: params.id },
      data: validated,
    });

    return NextResponse.json(updated);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          error: "Validation failed",
          issues: error.issues,
        },
        { status: 422 }
      );
    }

    console.error("[PATCH /api/items/[id]]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    // Verify ownership
    const item = await db.items.findFirst({
      where: {
        id: params.id,
        userId: auth.userId,
      },
    });

    if (!item) {
      return NextResponse.json({ error: "Not found" }, { status: 404 });
    }

    await db.items.delete({ where: { id: params.id } });

    return new NextResponse(null, { status: 204 });
  } catch (error) {
    console.error("[DELETE /api/items/[id]]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

---

## Recipe 3: Stripe Webhook Handler

Complete Stripe webhook processing with signature verification.

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { db } from "@/lib/db";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get("stripe-signature");

  if (!signature) {
    return NextResponse.json(
      { error: "Missing signature" },
      { status: 400 }
    );
  }

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (error) {
    console.error("Webhook signature verification failed:", error);
    return NextResponse.json(
      { error: "Invalid signature" },
      { status: 400 }
    );
  }

  try {
    switch (event.type) {
      case "payment_intent.succeeded": {
        const paymentIntent = event.data.object as Stripe.PaymentIntent;
        await handlePaymentSucceeded(paymentIntent);
        break;
      }

      case "payment_intent.payment_failed": {
        const paymentIntent = event.data.object as Stripe.PaymentIntent;
        await handlePaymentFailed(paymentIntent);
        break;
      }

      case "charge.dispute.created": {
        const dispute = event.data.object as Stripe.Dispute;
        await handleDispute(dispute);
        break;
      }

      case "customer.subscription.updated": {
        const subscription = event.data.object as Stripe.Subscription;
        await handleSubscriptionUpdated(subscription);
        break;
      }

      case "customer.subscription.deleted": {
        const subscription = event.data.object as Stripe.Subscription;
        await handleSubscriptionDeleted(subscription);
        break;
      }

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    console.error("Webhook processing error:", error);
    return NextResponse.json(
      { error: "Processing failed" },
      { status: 500 }
    );
  }
}

async function handlePaymentSucceeded(intent: Stripe.PaymentIntent) {
  const userId = intent.metadata?.userId;
  const orderId = intent.metadata?.orderId;

  if (!userId || !orderId) {
    console.warn("Missing metadata in payment intent");
    return;
  }

  // Update order status
  await db.orders.update({
    where: { id: orderId },
    data: {
      status: "paid",
      paidAt: new Date(),
      stripePaymentIntentId: intent.id,
    },
  });

  // Send confirmation email
  const user = await db.users.findUnique({ where: { id: userId } });
  if (user) {
    await sendConfirmationEmail(user.email, orderId);
  }
}

async function handlePaymentFailed(intent: Stripe.PaymentIntent) {
  const userId = intent.metadata?.userId;
  const orderId = intent.metadata?.orderId;

  if (!userId || !orderId) return;

  await db.orders.update({
    where: { id: orderId },
    data: {
      status: "payment_failed",
      failureReason: intent.last_payment_error?.message,
    },
  });

  const user = await db.users.findUnique({ where: { id: userId } });
  if (user) {
    await sendPaymentFailureEmail(user.email, orderId);
  }
}

async function handleDispute(dispute: Stripe.Dispute) {
  const chargeId = dispute.charge as string;

  await db.disputes.create({
    data: {
      stripeDisputeId: dispute.id,
      stripeChargeId: chargeId,
      status: dispute.status,
      reason: dispute.reason,
      amount: dispute.amount,
    },
  });

  // Notify support team
  await notifySupport(`Dispute created: ${dispute.id}`);
}

async function handleSubscriptionUpdated(sub: Stripe.Subscription) {
  const userId = sub.metadata?.userId;

  if (!userId) return;

  await db.subscriptions.update({
    where: { stripeSubscriptionId: sub.id },
    data: {
      status: sub.status,
      currentPeriodEnd: new Date(sub.current_period_end * 1000),
    },
  });
}

async function handleSubscriptionDeleted(sub: Stripe.Subscription) {
  const userId = sub.metadata?.userId;

  if (!userId) return;

  await db.subscriptions.update({
    where: { stripeSubscriptionId: sub.id },
    data: {
      status: "canceled",
      canceledAt: new Date(),
    },
  });
}

async function sendConfirmationEmail(email: string, orderId: string) {
  // Implementation
}

async function sendPaymentFailureEmail(email: string, orderId: string) {
  // Implementation
}

async function notifySupport(message: string) {
  // Implementation
}
```

---

## Recipe 4: Authenticated CRUD Endpoint

Full example with permission checks and audit logging.

```typescript
// app/api/projects/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { verifyAuth } from "@/lib/auth";
import { db } from "@/lib/db";

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const project = await db.projects.findUnique({
      where: { id: params.id },
      include: {
        members: {
          select: { userId: true, role: true },
        },
      },
    });

    if (!project) {
      return NextResponse.json({ error: "Not found" }, { status: 404 });
    }

    // Check if user has access
    const isMember = project.members.some((m) => m.userId === auth.userId);
    const isOwner = project.ownerId === auth.userId;

    if (!isMember && !isOwner) {
      return NextResponse.json(
        { error: "Forbidden" },
        { status: 403 }
      );
    }

    return NextResponse.json(project);
  } catch (error) {
    console.error("[GET /api/projects/[id]]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const project = await db.projects.findUnique({
      where: { id: params.id },
    });

    if (!project) {
      return NextResponse.json({ error: "Not found" }, { status: 404 });
    }

    // Only owner can update
    if (project.ownerId !== auth.userId) {
      return NextResponse.json(
        { error: "Forbidden" },
        { status: 403 }
      );
    }

    const body = await request.json();

    const updated = await db.projects.update({
      where: { id: params.id },
      data: {
        name: body.name,
        description: body.description,
        updatedAt: new Date(),
      },
    });

    // Audit log
    await db.auditLog.create({
      data: {
        userId: auth.userId,
        projectId: params.id,
        action: "project.updated",
        changes: JSON.stringify({
          from: project,
          to: updated,
        }),
      },
    });

    return NextResponse.json(updated);
  } catch (error) {
    console.error("[PATCH /api/projects/[id]]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const project = await db.projects.findUnique({
      where: { id: params.id },
    });

    if (!project) {
      return NextResponse.json({ error: "Not found" }, { status: 404 });
    }

    // Only owner can delete
    if (project.ownerId !== auth.userId) {
      return NextResponse.json(
        { error: "Forbidden" },
        { status: 403 }
      );
    }

    await db.projects.delete({ where: { id: params.id } });

    // Audit log
    await db.auditLog.create({
      data: {
        userId: auth.userId,
        projectId: params.id,
        action: "project.deleted",
      },
    });

    return new NextResponse(null, { status: 204 });
  } catch (error) {
    console.error("[DELETE /api/projects/[id]]", error);
    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

---

## Recipe 5: File Upload Endpoint

Secure file upload with validation and S3 integration.

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from "next/server";
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { verifyAuth } from "@/lib/auth";
import { db } from "@/lib/db";

const s3 = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const ALLOWED_TYPES = [
  "image/jpeg",
  "image/png",
  "image/webp",
  "application/pdf",
];
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

export async function POST(request: NextRequest) {
  try {
    const auth = await verifyAuth(request);

    if (!auth) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const formData = await request.formData();
    const file = formData.get("file") as File;

    if (!file) {
      return NextResponse.json(
        { error: "No file provided" },
        { status: 400 }
      );
    }

    // Validate file type
    if (!ALLOWED_TYPES.includes(file.type)) {
      return NextResponse.json(
        { error: "File type not allowed" },
        { status: 400 }
      );
    }

    // Validate file size
    if (file.size > MAX_FILE_SIZE) {
      return NextResponse.json(
        { error: "File too large" },
        { status: 413 }
      );
    }

    const fileBuffer = await file.arrayBuffer();
    const fileName = `${auth.userId}/${Date.now()}-${file.name}`;

    // Upload to S3
    await s3.send(
      new PutObjectCommand({
        Bucket: process.env.S3_BUCKET!,
        Key: fileName,
        Body: Buffer.from(fileBuffer),
        ContentType: file.type,
      })
    );

    const fileUrl = `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${fileName}`;

    // Record in database
    const uploadRecord = await db.uploads.create({
      data: {
        userId: auth.userId,
        fileName: file.name,
        fileSize: file.size,
        fileType: file.type,
        s3Key: fileName,
        url: fileUrl,
      },
    });

    return NextResponse.json(uploadRecord, { status: 201 });
  } catch (error) {
    console.error("[POST /api/upload]", error);
    return NextResponse.json(
      { error: "Upload failed" },
      { status: 500 }
    );
  }
}
```

---

## Recipe 6: Cron Job Edge Function

Scheduled task for cleanup, notifications, or syncing.

```typescript
// supabase/functions/daily-digest/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.0.0";

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
};

interface DigestEmail {
  userId: string;
  email: string;
  itemCount: number;
  lastDigestDate: string;
}

async function sendDigestEmails(
  supabase: ReturnType<typeof createClient>,
  emails: DigestEmail[]
) {
  const results = [];

  for (const { userId, email, itemCount, lastDigestDate } of emails) {
    try {
      // Generate digest content
      const { data: items } = await supabase
        .from("items")
        .select("*")
        .eq("user_id", userId)
        .gte("created_at", lastDigestDate)
        .limit(10);

      // Send email (using external service)
      const response = await fetch("https://api.sendgrid.com/v3/mail/send", {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${Deno.env.get("SENDGRID_API_KEY")}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          personalizations: [
            {
              to: [{ email }],
            },
          ],
          from: {
            email: "noreply@example.com",
            name: "Daily Digest",
          },
          subject: `Your Daily Digest - ${new Date().toLocaleDateString()}`,
          html: generateDigestHtml(items || [], itemCount),
        }),
      });

      if (response.ok) {
        // Update last digest timestamp
        await supabase
          .from("users")
          .update({ last_digest_at: new Date().toISOString() })
          .eq("id", userId);

        results.push({ userId, status: "sent" });
      } else {
        results.push({ userId, status: "failed", error: await response.text() });
      }
    } catch (error) {
      results.push({ userId, status: "error", error: error.message });
    }
  }

  return results;
}

function generateDigestHtml(
  items: any[],
  totalCount: number
): string {
  const itemsList = items
    .map(
      (item) =>
        `<li><strong>${item.name}</strong>: ${item.description || "No description"}</li>`
    )
    .join("");

  return `
    <h2>Your Daily Digest</h2>
    <p>You have <strong>${totalCount}</strong> new items today.</p>
    <h3>Recent Items:</h3>
    <ul>${itemsList}</ul>
    <p><a href="https://example.com/items">View all items</a></p>
  `;
}

async function handleRequest(req: Request): Promise<Response> {
  // Verify this is a legitimate cron call
  const secret = req.headers.get("authorization");
  const expectedSecret = `Bearer ${Deno.env.get("CRON_SECRET")}`;

  if (secret !== expectedSecret) {
    return new Response("Unauthorized", { status: 401, headers: corsHeaders });
  }

  try {
    const supabase = createClient(
      Deno.env.get("SUPABASE_URL")!,
      Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
    );

    // Find users who need digest emails
    const { data: users, error: usersError } = await supabase
      .from("users")
      .select("id, email, last_digest_at")
      .eq("digest_enabled", true)
      .lt(
        "last_digest_at",
        new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
      );

    if (usersError) throw usersError;

    if (!users || users.length === 0) {
      return new Response(
        JSON.stringify({
          success: true,
          message: "No users need digest emails",
        }),
        {
          status: 200,
          headers: { ...corsHeaders, "Content-Type": "application/json" },
        }
      );
    }

    // Get item counts and send emails
    const digestEmails: DigestEmail[] = [];

    for (const user of users) {
      const { count } = await supabase
        .from("items")
        .select("*", { count: "exact", head: true })
        .eq("user_id", user.id)
        .gte("created_at", user.last_digest_at || "1900-01-01");

      digestEmails.push({
        userId: user.id,
        email: user.email,
        itemCount: count || 0,
        lastDigestDate: user.last_digest_at || "1900-01-01",
      });
    }

    const results = await sendDigestEmails(supabase, digestEmails);

    const successful = results.filter((r) => r.status === "sent").length;

    return new Response(
      JSON.stringify({
        success: true,
        message: `Sent ${successful} digest emails`,
        results,
        timestamp: new Date().toISOString(),
      }),
      {
        status: 200,
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      }
    );
  } catch (error) {
    console.error("Cron job error:", error);
    return new Response(
      JSON.stringify({
        success: false,
        error: error.message,
        timestamp: new Date().toISOString(),
      }),
      {
        status: 500,
        headers: { ...corsHeaders, "Content-Type": "application/json" },
      }
    );
  }
}

serve(handleRequest);
```

---

## Recipe 7: Rate Limiter Middleware

Reusable rate limiting for API protection.

```typescript
// lib/rate-limit.ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";
import { NextRequest } from "next/server";

const redis = Redis.fromEnv();

// Different limiters for different use cases
const limiters = {
  api: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(100, "1 h"),
  }),
  auth: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, "15 m"),
  }),
  upload: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, "1 d"),
  }),
};

export async function checkRateLimit(
  request: NextRequest,
  type: keyof typeof limiters = "api",
  identifier?: string
): Promise<{ success: boolean; remaining: number; reset: Date }> {
  const ip =
    request.headers.get("x-forwarded-for") ||
    request.headers.get("x-real-ip") ||
    "unknown";

  const key = identifier || ip;
  const limiter = limiters[type];

  const { success, remaining, reset } = await limiter.limit(key);

  return {
    success,
    remaining: Math.max(0, remaining),
    reset: new Date(reset),
  };
}

// Usage in middleware
export async function rateLimit(
  request: NextRequest,
  type?: keyof typeof limiters,
  identifier?: string
) {
  const { success, remaining, reset } = await checkRateLimit(
    request,
    type,
    identifier
  );

  if (!success) {
    const response = new Response("Too many requests", { status: 429 });
    response.headers.set("Retry-After", reset.toISOString());
    response.headers.set("X-RateLimit-Remaining", "0");
    return response;
  }

  return null; // Continue
}
```

```typescript
// Usage in route handler
import { rateLimit } from "@/lib/rate-limit";

export async function POST(request: NextRequest) {
  const limitResponse = await rateLimit(request, "api");

  if (limitResponse) {
    return limitResponse;
  }

  // Continue with normal handler logic
}
```

---

## Deployment Commands

```bash
# Deploy Supabase Edge Function
supabase functions deploy my-function

# Deploy Next.js API
npm run build
npm run start

# Check function logs
supabase functions fetch-logs my-function --limit 100

# Test edge function locally
supabase functions serve

# Deploy from CI/CD
supabase projects deploy-config
```
