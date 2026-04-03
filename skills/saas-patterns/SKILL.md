---
name: saas-patterns
description: >
  Production SaaS patterns for the Prolific stack: Next.js, Supabase, and Stripe.
  Covers authentication, billing, multi-tenancy, realtime features, email, onboarding,
  and launch-ready checklist. Build scalable, secure SaaS applications with proven patterns
  and production-grade code examples.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Next.js, Supabase Documentation, Stripe API, Resend Email API, production SaaS experience"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# SaaS Patterns: The Prolific Stack Reference

Production-ready patterns for building scalable SaaS applications with Next.js, Supabase, and Stripe. This guide covers everything you need before launch: authentication, billing, multi-tenancy, realtime collaboration, email infrastructure, and user onboarding.

---

## 1. Authentication & User Management

### 1.1 Supabase Auth Configuration

Set up Supabase Auth with email/password and OAuth providers. Configure in your `supabase/config.toml`:

```toml
[auth]
enable_signup = true
email_autoconfirm = false
phone_autoconfirm = false

[auth.email]
enable_signup = true
enable_confirmations = true
confirm_email_before_using = true

[auth.external.google]
enabled = true
client_id = "env(GOOGLE_CLIENT_ID)"
secret = "env(GOOGLE_CLIENT_SECRET)"

[auth.external.github]
enabled = true
client_id = "env(GITHUB_CLIENT_ID)"
secret = "env(GITHUB_CLIENT_SECRET)"
```

### 1.2 Auth Middleware (Next.js App Router)

Create `middleware.ts` in your project root for route protection:

```typescript
import { createServerClient } from "@supabase/ssr";
import { NextResponse, type NextRequest } from "next/server";

const publicRoutes = ["/login", "/signup", "/reset-password", "/"];

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Allow public routes
  if (publicRoutes.includes(pathname)) {
    return NextResponse.next();
  }

  // Create Supabase client
  let response = NextResponse.next({
    request: {
      headers: request.headers,
    },
  });

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll();
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) =>
            response.cookies.set(name, value, options)
          );
        },
      },
    }
  );

  const { data } = await supabase.auth.getSession();

  if (!data.session) {
    const redirectUrl = request.nextUrl.clone();
    redirectUrl.pathname = "/login";
    redirectUrl.searchParams.set("redirect_to", pathname);
    return NextResponse.redirect(redirectUrl);
  }

  return response;
}

export const config = {
  matcher: [
    "/((?!_next/static|_next/image|favicon.ico).*)",
  ],
};
```

### 1.3 Auth Callback Handler

Create `app/auth/callback/route.ts` to handle OAuth redirects:

```typescript
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const code = searchParams.get("code");
  const redirectTo = searchParams.get("redirect_to") || "/dashboard";

  if (code) {
    const cookieStore = await cookies();
    const supabase = createServerClient(
      process.env.NEXT_PUBLIC_SUPABASE_URL,
      process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
      {
        cookies: {
          getAll() {
            return cookieStore.getAll();
          },
          setAll(cookiesToSet) {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            );
          },
        },
      }
    );

    const { error } = await supabase.auth.exchangeCodeForSession(code);

    if (!error) {
      return NextResponse.redirect(new URL(redirectTo, request.url));
    }
  }

  return NextResponse.redirect(new URL("/login?error=auth_failed", request.url));
}
```

### 1.4 User Profile Creation on Signup

Create a trigger in Supabase to auto-create user profiles:

```sql
-- Enable UUID extension
create extension if not exists "uuid-ossp" schema extensions;

-- Create profiles table
create table public.profiles (
  id uuid primary key references auth.users on delete cascade,
  email text unique,
  full_name text,
  avatar_url text,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

-- Enable RLS
alter table public.profiles enable row level security;

-- Allow users to read their own profile
create policy "Users can view their own profile" on public.profiles
  for select using (auth.uid() = id);

-- Allow users to update their own profile
create policy "Users can update their own profile" on public.profiles
  for update using (auth.uid() = id);

-- Create user profile on auth signup
create function public.handle_new_user()
returns trigger as $$
begin
  insert into public.profiles (id, email, full_name)
  values (new.id, new.email, new.user_metadata->>'full_name');
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure public.handle_new_user();
```

### 1.5 Auth Hooks for React

Create `lib/hooks/useAuth.ts`:

```typescript
import { useEffect, useState } from "react";
import { Session } from "@supabase/supabase-js";
import { createClient } from "@/lib/supabase/client";

export function useAuth() {
  const [session, setSession] = useState<Session | null>(null);
  const [loading, setLoading] = useState(true);
  const supabase = createClient();

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
      setLoading(false);
    });

    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange((_event, session) => {
      setSession(session);
      setLoading(false);
    });

    return () => subscription?.unsubscribe();
  }, []);

  return { session, loading };
}
```

---

## 2. Stripe Integration & Billing

### 2.1 Stripe Products and Prices

Set up products in Stripe Dashboard or use the Stripe CLI. Define pricing tiers:

```
Product: Pro
- $29/month (monthly_pro)
- $290/year (yearly_pro)

Product: Enterprise
- Custom pricing (contact sales)
```

### 2.2 Stripe Webhook Handler

Create `app/api/webhooks/stripe/route.ts`:

```typescript
import { stripe } from "@/lib/stripe";
import { createClient } from "@/lib/supabase/server";
import { headers } from "next/headers";

export async function POST(request: Request) {
  const body = await request.text();
  const headersList = await headers();
  const signature = headersList.get("stripe-signature");

  let event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET
    );
  } catch (err) {
    return new Response(
      `Webhook Error: ${err instanceof Error ? err.message : "Unknown error"}`,
      { status: 400 }
    );
  }

  const supabase = createClient();

  switch (event.type) {
    case "checkout.session.completed": {
      const session = event.data.object;
      const metadata = session.metadata;

      await supabase.from("subscriptions").insert({
        user_id: metadata.user_id,
        stripe_customer_id: session.customer,
        stripe_subscription_id: session.subscription,
        price_id: session.line_items[0].price.id,
        status: "active",
      });
      break;
    }

    case "invoice.paid": {
      const invoice = event.data.object;

      await supabase
        .from("subscriptions")
        .update({ status: "active" })
        .eq("stripe_customer_id", invoice.customer);
      break;
    }

    case "customer.subscription.updated": {
      const subscription = event.data.object;

      const updateData: Record<string, unknown> = {
        status: subscription.status,
      };

      if (subscription.items.data[0]) {
        updateData.price_id = subscription.items.data[0].price.id;
      }

      await supabase
        .from("subscriptions")
        .update(updateData)
        .eq("stripe_subscription_id", subscription.id);
      break;
    }

    case "customer.subscription.deleted": {
      const subscription = event.data.object;

      await supabase
        .from("subscriptions")
        .update({ status: "canceled" })
        .eq("stripe_subscription_id", subscription.id);
      break;
    }

    case "customer.created": {
      const customer = event.data.object;

      await supabase
        .from("customers")
        .insert({
          stripe_customer_id: customer.id,
          email: customer.email,
        })
        .onConflict("stripe_customer_id");
      break;
    }

    default:
      console.log(`Unhandled event type ${event.type}`);
  }

  return new Response(JSON.stringify({ received: true }), { status: 200 });
}
```

### 2.3 Checkout Session Creation

Create `lib/stripe/checkout.ts`:

```typescript
import { stripe } from "@/lib/stripe";

export async function createCheckoutSession({
  userId,
  priceId,
  customerId,
  successUrl,
  cancelUrl,
}: {
  userId: string;
  priceId: string;
  customerId?: string;
  successUrl: string;
  cancelUrl: string;
}) {
  let stripeCustomerId = customerId;

  // Create or retrieve customer
  if (!stripeCustomerId) {
    const customer = await stripe.customers.create({
      metadata: { user_id: userId },
    });
    stripeCustomerId = customer.id;
  }

  const session = await stripe.checkout.sessions.create({
    customer: stripeCustomerId,
    line_items: [
      {
        price: priceId,
        quantity: 1,
      },
    ],
    mode: "subscription",
    success_url: successUrl,
    cancel_url: cancelUrl,
    metadata: { user_id: userId },
  });

  return session;
}
```

### 2.4 Customer Portal

Create `app/api/billing/portal/route.ts`:

```typescript
import { stripe } from "@/lib/stripe";
import { createClient } from "@/lib/supabase/server";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const supabase = createClient();

  const {
    data: { user },
  } = await supabase.auth.getUser();

  if (!user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const { data: subscription } = await supabase
    .from("subscriptions")
    .select("stripe_customer_id")
    .eq("user_id", user.id)
    .single();

  if (!subscription?.stripe_customer_id) {
    return NextResponse.json(
      { error: "No active subscription" },
      { status: 400 }
    );
  }

  const portalSession = await stripe.billingPortal.sessions.create({
    customer: subscription.stripe_customer_id,
    return_url: `${request.nextUrl.origin}/billing`,
  });

  return NextResponse.json({ url: portalSession.url });
}
```

---

## 3. Multi-Tenancy & Organizations

### 3.1 Organizations Table Schema

```sql
create table public.organizations (
  id uuid primary key default uuid_generate_v4(),
  name text not null,
  slug text unique not null,
  owner_id uuid references public.profiles not null,
  avatar_url text,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

create table public.org_members (
  id uuid primary key default uuid_generate_v4(),
  org_id uuid references public.organizations on delete cascade,
  user_id uuid references public.profiles on delete cascade,
  role text not null default 'member',
  created_at timestamp with time zone default now(),
  unique(org_id, user_id)
);

alter table public.organizations enable row level security;
alter table public.org_members enable row level security;

-- Org members can view their organizations
create policy "Users can view their organizations" on public.organizations
  for select using (exists(
    select 1 from public.org_members
    where org_members.org_id = organizations.id
    and org_members.user_id = auth.uid()
  ));

-- Org members can view other members in their org
create policy "Org members can view org members" on public.org_members
  for select using (
    user_id = auth.uid() or
    org_id in (
      select org_id from public.org_members
      where user_id = auth.uid()
    )
  );
```

### 3.2 Invite Flow

Create `app/api/orgs/[orgId]/invite/route.ts`:

```typescript
import { createClient } from "@/lib/supabase/server";
import { NextRequest, NextResponse } from "next/server";

export async function POST(
  request: NextRequest,
  { params }: { params: { orgId: string } }
) {
  const supabase = createClient();
  const { email, role = "member" } = await request.json();

  const {
    data: { user },
  } = await supabase.auth.getUser();

  // Check if user is org owner/admin
  const { data: member } = await supabase
    .from("org_members")
    .select("role")
    .eq("org_id", params.orgId)
    .eq("user_id", user.id)
    .single();

  if (!member || !["owner", "admin"].includes(member.role)) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 403 });
  }

  // Find or create user
  const { data: inviteUser } = await supabase
    .from("profiles")
    .select("id")
    .eq("email", email)
    .single();

  if (inviteUser) {
    // Add existing user to org
    await supabase.from("org_members").insert({
      org_id: params.orgId,
      user_id: inviteUser.id,
      role,
    });
  } else {
    // Send invite email for new user
    await supabase.auth.admin.inviteUserByEmail(email, {
      redirectTo: `${request.nextUrl.origin}/signup?invite_org=${params.orgId}&role=${role}`,
    });
  }

  return NextResponse.json({ success: true });
}
```

### 3.3 Organization Switcher Hook

Create `lib/hooks/useOrgSwitcher.ts`:

```typescript
import { useCallback, useEffect, useState } from "react";
import { createClient } from "@/lib/supabase/client";

interface Org {
  id: string;
  name: string;
  slug: string;
}

export function useOrgSwitcher() {
  const [organizations, setOrganizations] = useState<Org[]>([]);
  const [currentOrg, setCurrentOrg] = useState<Org | null>(null);
  const supabase = createClient();

  useEffect(() => {
    loadOrganizations();
  }, []);

  async function loadOrganizations() {
    const {
      data: { user },
    } = await supabase.auth.getUser();

    if (!user) return;

    const { data } = await supabase
      .from("organizations")
      .select("id, name, slug")
      .order("name");

    setOrganizations(data || []);
    if (data && data.length > 0) {
      setCurrentOrg(data[0]);
    }
  }

  const switchOrg = useCallback((org: Org) => {
    setCurrentOrg(org);
  }, []);

  return {
    organizations,
    currentOrg,
    switchOrg,
    reloadOrganizations: loadOrganizations,
  };
}
```

---

## 4. Realtime Features

### 4.1 Realtime Channel Setup

Create `lib/realtime/channels.ts`:

```typescript
import { createClient } from "@/lib/supabase/client";

const supabase = createClient();

// Presence: Track who's online
export function subscribeToPresence(room: string, callback: (state: any) => void) {
  const channel = supabase.channel(`presence:${room}`, {
    config: { presence: { key: room } },
  });

  channel.on("presence", { event: "sync" }, () => {
    const state = channel.presenceState();
    callback(state);
  });

  channel.subscribe((status) => {
    if (status === "SUBSCRIBED") {
      channel.track({
        user_id: "current_user_id",
        online_at: new Date().toISOString(),
      });
    }
  });

  return channel;
}

// Broadcast: Send real-time messages
export function broadcastMessage(room: string, event: string, payload: any) {
  const channel = supabase.channel(room);

  channel.subscribe((status) => {
    if (status === "SUBSCRIBED") {
      channel.send({
        type: "broadcast",
        event,
        payload,
      });
    }
  });
}

// Listen to Postgres changes
export function subscribeToChanges(
  table: string,
  callback: (payload: any) => void
) {
  return supabase
    .channel(`realtime:${table}`)
    .on(
      "postgres_changes",
      {
        event: "*",
        schema: "public",
        table,
      },
      callback
    )
    .subscribe();
}
```

### 4.2 React Hook for Realtime Subscriptions

Create `lib/hooks/useRealtimeSubscription.ts`:

```typescript
import { useEffect, useState } from "react";
import { createClient } from "@/lib/supabase/client";

export function useRealtimeSubscription<T>(
  table: string,
  query?: Record<string, any>
) {
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);
  const supabase = createClient();

  useEffect(() => {
    // Initial fetch
    supabase
      .from(table)
      .select()
      .then(({ data }) => {
        setData(data || []);
        setLoading(false);
      });

    // Subscribe to changes
    const subscription = supabase
      .channel(`realtime:${table}`)
      .on(
        "postgres_changes",
        {
          event: "*",
          schema: "public",
          table,
        },
        (payload) => {
          if (payload.eventType === "INSERT") {
            setData((prev) => [payload.new as T, ...prev]);
          } else if (payload.eventType === "UPDATE") {
            setData((prev) =>
              prev.map((item: any) =>
                item.id === payload.new.id ? (payload.new as T) : item
              )
            );
          } else if (payload.eventType === "DELETE") {
            setData((prev) =>
              prev.filter((item: any) => item.id !== payload.old.id)
            );
          }
        }
      )
      .subscribe();

    return () => {
      subscription.unsubscribe();
    };
  }, [table]);

  return { data, loading };
}
```

---

## 5. Email Infrastructure with Resend

### 5.1 Transactional Email Templates

Create `lib/email/templates.ts`:

```typescript
export const emailTemplates = {
  welcome: (name: string, appUrl: string) => ({
    subject: "Welcome to [YourApp]",
    html: `
      <h1>Welcome, ${name}!</h1>
      <p>Thanks for signing up. Get started here:</p>
      <a href="${appUrl}/onboarding">Start Setup</a>
    `,
  }),

  invoice: (invoiceUrl: string, amount: string, dueDate: string) => ({
    subject: "Your invoice is ready",
    html: `
      <h2>Invoice</h2>
      <p>Amount due: $${amount}</p>
      <p>Due date: ${dueDate}</p>
      <a href="${invoiceUrl}">View Invoice</a>
    `,
  }),

  passwordReset: (resetUrl: string) => ({
    subject: "Reset your password",
    html: `
      <p>Click below to reset your password. This link expires in 1 hour.</p>
      <a href="${resetUrl}">Reset Password</a>
    `,
  }),

  subscriptionConfirm: (plan: string, renewDate: string) => ({
    subject: "Subscription Confirmed",
    html: `
      <h2>Subscription Activated</h2>
      <p>Plan: ${plan}</p>
      <p>Renews: ${renewDate}</p>
    `,
  }),
};
```

### 5.2 Email Service Wrapper

Create `lib/email/send.ts`:

```typescript
import { Resend } from "resend";

const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendEmail({
  to,
  subject,
  html,
}: {
  to: string;
  subject: string;
  html: string;
}) {
  try {
    const result = await resend.emails.send({
      from: "noreply@yourapp.com",
      to,
      subject,
      html,
    });

    return result;
  } catch (error) {
    console.error("Email send error:", error);
    throw error;
  }
}

export async function sendWelcomeEmail(email: string, name: string) {
  const template = emailTemplates.welcome(name, process.env.NEXT_PUBLIC_APP_URL);
  return sendEmail({ to: email, ...template });
}
```

### 5.3 Notification Preferences

Create tables for email preferences:

```sql
create table public.notification_preferences (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.profiles on delete cascade,
  email_marketing boolean default true,
  email_invoices boolean default true,
  email_product_updates boolean default true,
  created_at timestamp with time zone default now(),
  unique(user_id)
);

alter table public.notification_preferences enable row level security;

create policy "Users can manage their preferences" on public.notification_preferences
  for all using (user_id = auth.uid());
```

---

## 6. Onboarding Experience

### 6.1 Progressive Onboarding Flow

Create `app/onboarding/page.tsx`:

```typescript
"use client";

import { useState } from "react";
import { createClient } from "@/lib/supabase/client";

const steps = ["profile", "organization", "team", "complete"];

export default function OnboardingPage() {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState({
    full_name: "",
    org_name: "",
    team_size: "",
  });
  const supabase = createClient();

  const handleNext = async () => {
    if (currentStep === 0) {
      // Save profile
      const { data: { user } } = await supabase.auth.getUser();
      await supabase
        .from("profiles")
        .update({ full_name: formData.full_name })
        .eq("id", user?.id);
    }

    if (currentStep === 1) {
      // Create organization
      const { data: { user } } = await supabase.auth.getUser();
      await supabase.from("organizations").insert({
        name: formData.org_name,
        slug: formData.org_name.toLowerCase().replace(/\s+/g, "-"),
        owner_id: user?.id,
      });
    }

    setCurrentStep((prev) => Math.min(prev + 1, steps.length - 1));
  };

  return (
    <div className="max-w-2xl mx-auto p-8">
      {currentStep === 0 && (
        <div>
          <h2>Welcome! Let's set up your profile.</h2>
          <input
            value={formData.full_name}
            onChange={(e) =>
              setFormData({ ...formData, full_name: e.target.value })
            }
            placeholder="Full name"
          />
        </div>
      )}

      {currentStep === 1 && (
        <div>
          <h2>Create your organization</h2>
          <input
            value={formData.org_name}
            onChange={(e) =>
              setFormData({ ...formData, org_name: e.target.value })
            }
            placeholder="Organization name"
          />
        </div>
      )}

      {currentStep === steps.length - 1 && (
        <div>
          <h2>All set!</h2>
          <p>Your account is ready. Start exploring.</p>
        </div>
      )}

      <button onClick={handleNext} className="mt-6">
        {currentStep === steps.length - 1 ? "Go to Dashboard" : "Next"}
      </button>
    </div>
  );
}
```

### 6.2 Empty State Patterns

```typescript
export function EmptyStateWithAction({ title, description, action }: any) {
  return (
    <div className="text-center py-12">
      <svg className="mx-auto h-12 w-12 text-gray-400" fill="none" stroke="currentColor">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
      </svg>
      <h3 className="mt-2 text-sm font-medium text-gray-900">{title}</h3>
      <p className="mt-1 text-sm text-gray-500">{description}</p>
      <button className="mt-6 inline-flex items-center px-4 py-2 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700">
        {action.label}
      </button>
    </div>
  );
}
```

---

## 7. Billing Models

### 7.1 Free Trial Implementation

```sql
create table public.trial_usage (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.profiles on delete cascade,
  trial_started_at timestamp with time zone default now(),
  trial_expires_at timestamp with time zone,
  converted_to_paid boolean default false,
  created_at timestamp with time zone default now()
);

-- Check trial status
create function public.get_trial_status(user_id uuid)
returns jsonb as $$
declare
  trial record;
begin
  select * into trial from public.trial_usage where user_id = user_id;
  return jsonb_build_object(
    'is_trial', trial is not null,
    'days_remaining', ceil(extract(epoch from (trial.trial_expires_at - now())) / 86400),
    'expired', trial.trial_expires_at < now()
  );
end;
$$ language plpgsql;
```

### 7.2 Usage-Based Metering

Create `app/api/billing/usage/route.ts`:

```typescript
import { createClient } from "@/lib/supabase/server";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const supabase = createClient();
  const { metricId, value } = await request.json();

  const {
    data: { user },
  } = await supabase.auth.getUser();

  const { data: subscription } = await supabase
    .from("subscriptions")
    .select("stripe_subscription_id")
    .eq("user_id", user?.id)
    .single();

  if (!subscription) {
    return NextResponse.json({ error: "No subscription" }, { status: 400 });
  }

  // Record usage for metering
  await supabase.from("metered_usage").insert({
    user_id: user?.id,
    metric_id: metricId,
    value,
    recorded_at: new Date(),
  });

  return NextResponse.json({ success: true });
}
```

### 7.3 Upgrade/Downgrade Flow

```typescript
export async function changeSubscription({
  userId,
  newPriceId,
}: {
  userId: string;
  newPriceId: string;
}) {
  const supabase = createClient();

  const { data: subscription } = await supabase
    .from("subscriptions")
    .select("stripe_subscription_id")
    .eq("user_id", userId)
    .single();

  if (!subscription) {
    throw new Error("No active subscription");
  }

  const stripeSubscription = await stripe.subscriptions.retrieve(
    subscription.stripe_subscription_id
  );

  const updated = await stripe.subscriptions.update(
    subscription.stripe_subscription_id,
    {
      items: [
        {
          id: stripeSubscription.items.data[0].id,
          price: newPriceId,
        },
      ],
      proration_behavior: "create_prorations",
    }
  );

  return updated;
}
```

---

## 8. Admin Dashboard

### 8.1 User Management Page

Create `app/admin/users/page.tsx`:

```typescript
"use client";

import { useEffect, useState } from "react";
import { createClient } from "@/lib/supabase/client";

export default function UsersAdminPage() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const supabase = createClient();

  useEffect(() => {
    loadUsers();
  }, []);

  async function loadUsers() {
    const { data } = await supabase
      .from("profiles")
      .select("*, subscriptions(status, price_id)")
      .order("created_at", { ascending: false });

    setUsers(data || []);
    setLoading(false);
  }

  async function deactivateUser(userId: string) {
    await supabase.auth.admin.deleteUser(userId);
    setUsers((prev) => prev.filter((u: any) => u.id !== userId));
  }

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h1>Users</h1>
      <table>
        <thead>
          <tr>
            <th>Email</th>
            <th>Name</th>
            <th>Subscription</th>
            <th>Joined</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {users.map((user: any) => (
            <tr key={user.id}>
              <td>{user.email}</td>
              <td>{user.full_name}</td>
              <td>{user.subscriptions?.[0]?.status || "None"}</td>
              <td>{new Date(user.created_at).toLocaleDateString()}</td>
              <td>
                <button onClick={() => deactivateUser(user.id)}>
                  Deactivate
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### 8.2 Subscription Analytics

Create `app/admin/analytics/page.tsx`:

```typescript
"use client";

import { useEffect, useState } from "react";
import { createClient } from "@/lib/supabase/client";

export default function AnalyticsPage() {
  const [metrics, setMetrics] = useState({
    totalUsers: 0,
    activeSubscriptions: 0,
    mrr: 0,
    churnRate: 0,
  });
  const supabase = createClient();

  useEffect(() => {
    loadMetrics();
  }, []);

  async function loadMetrics() {
    // Total users
    const { count: totalUsers } = await supabase
      .from("profiles")
      .select("*", { count: "exact" });

    // Active subscriptions
    const { count: activeSubscriptions } = await supabase
      .from("subscriptions")
      .select("*", { count: "exact" })
      .eq("status", "active");

    setMetrics({
      totalUsers: totalUsers || 0,
      activeSubscriptions: activeSubscriptions || 0,
      mrr: 0, // Calculate from Stripe
      churnRate: 0, // Calculate from Stripe
    });
  }

  return (
    <div className="grid grid-cols-4 gap-4">
      <div className="p-4 bg-white rounded border">
        <h3>Total Users</h3>
        <p className="text-2xl font-bold">{metrics.totalUsers}</p>
      </div>
      <div className="p-4 bg-white rounded border">
        <h3>Active Subscriptions</h3>
        <p className="text-2xl font-bold">{metrics.activeSubscriptions}</p>
      </div>
      <div className="p-4 bg-white rounded border">
        <h3>MRR</h3>
        <p className="text-2xl font-bold">${metrics.mrr}</p>
      </div>
      <div className="p-4 bg-white rounded border">
        <h3>Churn Rate</h3>
        <p className="text-2xl font-bold">{metrics.churnRate}%</p>
      </div>
    </div>
  );
}
```

---

## 9. Production SaaS Launch Checklist

### Pre-Launch

- [ ] **Auth**: Supabase Auth configured, email verification enabled, OAuth providers connected
- [ ] **Database**: RLS policies enforced on all tables, backups enabled
- [ ] **Stripe**: Products/prices created, webhook endpoint configured and tested
- [ ] **Email**: Resend account created, sender domain verified, templates tested
- [ ] **CDN**: Next.js images optimized, assets cached globally
- [ ] **Monitoring**: Sentry/LogRocket configured, error tracking enabled
- [ ] **Security**: CORS configured, CSP headers set, rate limiting enabled
- [ ] **Compliance**: GDPR consent banner, privacy policy, terms of service reviewed by legal
- [ ] **Performance**: Lighthouse score > 90, Core Web Vitals optimized
- [ ] **SEO**: Meta tags, sitemap, robots.txt, Open Graph configured
- [ ] **DNS**: Custom domain configured, email MX records added

### At Launch

- [ ] **Monitoring**: Error tracking active, dashboard setup
- [ ] **Support**: Help desk configured, email routing verified
- [ ] **Analytics**: Plausible/Mixpanel configured, events tracked
- [ ] **Backups**: Database backup schedule confirmed
- [ ] **Incident Response**: On-call schedule, runbook created
- [ ] **Communication**: Status page, status email list ready

### Post-Launch

- [ ] **User Onboarding**: Track completion rates, iterate on flows
- [ ] **Billing**: Monitor payment failures, failed charge recovery
- [ ] **Support**: Track ticket volume, response times
- [ ] **Performance**: Monitor Core Web Vitals, optimize slow queries
- [ ] **Security**: Regular penetration testing, dependency updates
- [ ] **Growth**: Iterate on pricing, upsell flows, churn analysis

---

## Key Patterns Summary

1. **Auth**: Middleware protection, profile auto-creation, session management
2. **Billing**: Webhook sync to Supabase, subscription status tracking, pricing tiers
3. **Multi-Tenancy**: Org memberships, RLS policies, role-based access
4. **Realtime**: Presence, broadcast, Postgres change listeners
5. **Email**: Transactional via Resend, notification preferences
6. **Onboarding**: Progressive disclosure, empty states, feature tours
7. **Operations**: Admin dashboard, analytics, monitoring

This is production-ready. Use it.

<sub>Open RX by CutTheChexx — The Prescription.</sub>
