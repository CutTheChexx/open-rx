# Stripe Integration Recipes

Production-grade Stripe integration code for SaaS applications. Copy, customize, deploy.

---

## 1. Checkout Session Creation

### Basic Subscription Checkout

```typescript
import { stripe } from "@/lib/stripe";

export async function createSubscriptionCheckout({
  userId,
  email,
  priceId,
  successUrl,
  cancelUrl,
  customerData = {},
}: {
  userId: string;
  email: string;
  priceId: string;
  successUrl: string;
  cancelUrl: string;
  customerData?: Record<string, any>;
}) {
  const session = await stripe.checkout.sessions.create({
    customer_email: email,
    line_items: [
      {
        price: priceId,
        quantity: 1,
      },
    ],
    mode: "subscription",
    success_url: `${successUrl}?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: cancelUrl,
    metadata: {
      user_id: userId,
      ...customerData,
    },
    allow_promotion_codes: true,
    billing_address_collection: "required",
  });

  return session;
}
```

### One-Time Payment Checkout

```typescript
export async function createOneTimeCheckout({
  userId,
  email,
  amount,
  description,
  successUrl,
  cancelUrl,
}: {
  userId: string;
  email: string;
  amount: number; // in cents
  description: string;
  successUrl: string;
  cancelUrl: string;
}) {
  const session = await stripe.checkout.sessions.create({
    customer_email: email,
    line_items: [
      {
        price_data: {
          currency: "usd",
          product_data: {
            name: description,
          },
          unit_amount: amount,
        },
        quantity: 1,
      },
    ],
    mode: "payment",
    success_url: successUrl,
    cancel_url: cancelUrl,
    metadata: {
      user_id: userId,
    },
  });

  return session;
}
```

### Trial Subscription with Promo Code

```typescript
export async function createTrialCheckout({
  userId,
  email,
  priceId,
  trialDays = 14,
  successUrl,
  cancelUrl,
}: {
  userId: string;
  email: string;
  priceId: string;
  trialDays?: number;
  successUrl: string;
  cancelUrl: string;
}) {
  // First create/get customer
  const customer = await stripe.customers.create({
    email,
    metadata: { user_id: userId },
  });

  // Create subscription directly with trial
  const subscription = await stripe.subscriptions.create({
    customer: customer.id,
    items: [{ price: priceId }],
    trial_period_days: trialDays,
    metadata: { user_id: userId },
    expand: ["latest_invoice"],
  });

  return {
    subscriptionId: subscription.id,
    customerId: customer.id,
  };
}
```

---

## 2. Complete Webhook Handler

### Full Event Switch Statement

```typescript
import { stripe } from "@/lib/stripe";
import { createClient } from "@/lib/supabase/server";
import { headers } from "next/headers";
import { sendEmail } from "@/lib/email/send";

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

  try {
    switch (event.type) {
      // ==================
      // CUSTOMER EVENTS
      // ==================
      case "customer.created": {
        const customer = event.data.object;
        console.log("Customer created:", customer.id);

        await supabase.from("stripe_customers").insert({
          stripe_customer_id: customer.id,
          email: customer.email,
          metadata: customer.metadata,
        });
        break;
      }

      case "customer.updated": {
        const customer = event.data.object;
        console.log("Customer updated:", customer.id);

        await supabase
          .from("stripe_customers")
          .update({
            email: customer.email,
            metadata: customer.metadata,
          })
          .eq("stripe_customer_id", customer.id);
        break;
      }

      case "customer.deleted": {
        const customer = event.data.object;
        console.log("Customer deleted:", customer.id);

        await supabase
          .from("stripe_customers")
          .delete()
          .eq("stripe_customer_id", customer.id);
        break;
      }

      // ==================
      // CHECKOUT SESSION EVENTS
      // ==================
      case "checkout.session.completed": {
        const session = event.data.object;
        console.log("Checkout session completed:", session.id);

        const metadata = session.metadata || {};
        const customerId = session.customer as string;

        // Handle subscription mode
        if (session.mode === "subscription") {
          // Get subscription details from Stripe
          const subscription = await stripe.subscriptions.retrieve(
            session.subscription as string
          );

          const priceId = subscription.items.data[0].price.id;

          // Create subscription record in Supabase
          await supabase.from("subscriptions").upsert({
            user_id: metadata.user_id,
            stripe_customer_id: customerId,
            stripe_subscription_id: subscription.id,
            price_id: priceId,
            status: subscription.status,
            current_period_start: new Date(
              subscription.current_period_start * 1000
            ),
            current_period_end: new Date(
              subscription.current_period_end * 1000
            ),
          });

          // Send confirmation email
          if (session.customer_email) {
            await sendEmail({
              to: session.customer_email,
              subject: "Subscription Confirmed",
              html: "<p>Your subscription is now active. Check your dashboard.</p>",
            });
          }
        }

        // Handle payment mode (one-time)
        if (session.mode === "payment") {
          // Record payment
          await supabase.from("payments").insert({
            user_id: metadata.user_id,
            stripe_session_id: session.id,
            amount: session.amount_total,
            currency: session.currency,
            status: "completed",
          });
        }
        break;
      }

      case "checkout.session.expired": {
        const session = event.data.object;
        console.log("Checkout session expired:", session.id);
        // Optional: Send "come back" email
        break;
      }

      // ==================
      // SUBSCRIPTION EVENTS
      // ==================
      case "customer.subscription.created": {
        const subscription = event.data.object;
        console.log("Subscription created:", subscription.id);

        await supabase.from("subscriptions").insert({
          stripe_subscription_id: subscription.id,
          stripe_customer_id: subscription.customer as string,
          price_id: subscription.items.data[0].price.id,
          status: subscription.status,
          current_period_start: new Date(
            subscription.current_period_start * 1000
          ),
          current_period_end: new Date(subscription.current_period_end * 1000),
        });
        break;
      }

      case "customer.subscription.updated": {
        const subscription = event.data.object;
        console.log("Subscription updated:", subscription.id);

        const updateData: Record<string, any> = {
          status: subscription.status,
          current_period_start: new Date(
            subscription.current_period_start * 1000
          ),
          current_period_end: new Date(subscription.current_period_end * 1000),
        };

        // Track plan changes
        if (subscription.items.data[0]) {
          updateData.price_id = subscription.items.data[0].price.id;
        }

        // Handle subscription pause
        if (subscription.pause_collection) {
          updateData.paused = true;
          updateData.paused_at = new Date();
        } else if (subscription.pause_collection === null) {
          updateData.paused = false;
        }

        await supabase
          .from("subscriptions")
          .update(updateData)
          .eq("stripe_subscription_id", subscription.id);

        // Send update notification
        const customer = await stripe.customers.retrieve(
          subscription.customer as string
        );
        if (customer && typeof customer.email === "string") {
          await sendEmail({
            to: customer.email,
            subject: "Your subscription has been updated",
            html: `<p>Your subscription to ${subscription.items.data[0].price.nickname} is now active.</p>`,
          });
        }
        break;
      }

      case "customer.subscription.deleted": {
        const subscription = event.data.object;
        console.log("Subscription deleted:", subscription.id);

        // Update subscription status
        await supabase
          .from("subscriptions")
          .update({
            status: "canceled",
            canceled_at: new Date(),
          })
          .eq("stripe_subscription_id", subscription.id);

        // Send cancellation email
        const customer = await stripe.customers.retrieve(
          subscription.customer as string
        );
        if (customer && typeof customer.email === "string") {
          await sendEmail({
            to: customer.email,
            subject: "Your subscription has been canceled",
            html: "<p>We're sorry to see you go. Your subscription is now canceled.</p>",
          });
        }
        break;
      }

      case "customer.subscription.trial_will_end": {
        const subscription = event.data.object;
        console.log("Trial ending in 3 days:", subscription.id);

        // Send reminder email
        const customer = await stripe.customers.retrieve(
          subscription.customer as string
        );
        if (customer && typeof customer.email === "string") {
          await sendEmail({
            to: customer.email,
            subject: "Your free trial is ending soon",
            html: "<p>Your trial ends in 3 days. Add a payment method to continue.</p>",
          });
        }
        break;
      }

      // ==================
      // INVOICE EVENTS
      // ==================
      case "invoice.created": {
        const invoice = event.data.object;
        console.log("Invoice created:", invoice.id);

        await supabase.from("invoices").insert({
          stripe_invoice_id: invoice.id,
          stripe_customer_id: invoice.customer as string,
          amount: invoice.amount_paid,
          amount_due: invoice.amount_due,
          currency: invoice.currency,
          status: invoice.status,
          invoice_url: invoice.hosted_invoice_url,
          pdf_url: invoice.invoice_pdf,
        });
        break;
      }

      case "invoice.finalized": {
        const invoice = event.data.object;
        console.log("Invoice finalized:", invoice.id);

        await supabase
          .from("invoices")
          .update({
            status: "open",
            finalized_at: new Date(),
          })
          .eq("stripe_invoice_id", invoice.id);

        // Send invoice email
        const customer = await stripe.customers.retrieve(
          invoice.customer as string
        );
        if (customer && typeof customer.email === "string") {
          await sendEmail({
            to: customer.email,
            subject: "Your invoice is ready",
            html: `
              <p>Your invoice for $${(invoice.amount_due / 100).toFixed(2)} is ready.</p>
              <a href="${invoice.hosted_invoice_url}">View Invoice</a>
            `,
          });
        }
        break;
      }

      case "invoice.payment_succeeded": {
        const invoice = event.data.object;
        console.log("Invoice payment succeeded:", invoice.id);

        await supabase
          .from("invoices")
          .update({
            status: "paid",
            paid_at: new Date(invoice.status_transitions.paid_at * 1000),
          })
          .eq("stripe_invoice_id", invoice.id);

        // Update subscription status
        if (invoice.subscription) {
          await supabase
            .from("subscriptions")
            .update({ status: "active" })
            .eq("stripe_subscription_id", invoice.subscription);
        }
        break;
      }

      case "invoice.payment_failed": {
        const invoice = event.data.object;
        console.log("Invoice payment failed:", invoice.id);

        await supabase
          .from("invoices")
          .update({
            status: "open", // Still open, awaiting retry
            last_payment_error: invoice.last_payment_error?.message,
          })
          .eq("stripe_invoice_id", invoice.id);

        // Send payment failed email with retry information
        const customer = await stripe.customers.retrieve(
          invoice.customer as string
        );
        if (customer && typeof customer.email === "string") {
          await sendEmail({
            to: customer.email,
            subject: "Payment Failed — Action Required",
            html: `
              <p>We couldn't process your payment.</p>
              <p>Reason: ${invoice.last_payment_error?.message}</p>
              <p>Please update your payment method:</p>
              <a href="${process.env.NEXT_PUBLIC_APP_URL}/billing">Update Payment</a>
            `,
          });
        }
        break;
      }

      case "invoice.voided": {
        const invoice = event.data.object;
        console.log("Invoice voided:", invoice.id);

        await supabase
          .from("invoices")
          .update({ status: "void" })
          .eq("stripe_invoice_id", invoice.id);
        break;
      }

      // ==================
      // PAYMENT INTENT EVENTS
      // ==================
      case "payment_intent.succeeded": {
        const paymentIntent = event.data.object;
        console.log("Payment intent succeeded:", paymentIntent.id);

        await supabase.from("payments").insert({
          stripe_payment_intent_id: paymentIntent.id,
          amount: paymentIntent.amount,
          currency: paymentIntent.currency,
          status: "succeeded",
          metadata: paymentIntent.metadata,
        });
        break;
      }

      case "payment_intent.payment_failed": {
        const paymentIntent = event.data.object;
        console.log("Payment intent failed:", paymentIntent.id);

        await supabase.from("payments").insert({
          stripe_payment_intent_id: paymentIntent.id,
          amount: paymentIntent.amount,
          currency: paymentIntent.currency,
          status: "failed",
          error_message: paymentIntent.last_payment_error?.message,
          metadata: paymentIntent.metadata,
        });
        break;
      }

      // ==================
      // CHARGE EVENTS
      // ==================
      case "charge.succeeded": {
        const charge = event.data.object;
        console.log("Charge succeeded:", charge.id);

        // Payment has gone through
        break;
      }

      case "charge.refunded": {
        const charge = event.data.object;
        console.log("Charge refunded:", charge.id);

        await supabase
          .from("payments")
          .update({ status: "refunded" })
          .eq("stripe_charge_id", charge.id);
        break;
      }

      case "charge.dispute.created": {
        const charge = event.data.object;
        console.log("Charge dispute created:", charge.id);

        // Alert admin about chargeback
        await sendEmail({
          to: "admin@yourapp.com",
          subject: "Chargeback Alert",
          html: `<p>Charge ${charge.id} has been disputed.</p>`,
        });
        break;
      }

      // ==================
      // COUPON & PROMO EVENTS
      // ==================
      case "coupon.created": {
        const coupon = event.data.object;
        console.log("Coupon created:", coupon.id);

        await supabase.from("coupons").insert({
          stripe_coupon_id: coupon.id,
          percent_off: coupon.percent_off,
          amount_off: coupon.amount_off,
          duration: coupon.duration,
          valid: !coupon.deleted,
        });
        break;
      }

      case "coupon.deleted": {
        const coupon = event.data.object;
        console.log("Coupon deleted:", coupon.id);

        await supabase
          .from("coupons")
          .update({ valid: false })
          .eq("stripe_coupon_id", coupon.id);
        break;
      }

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return new Response(JSON.stringify({ received: true }), { status: 200 });
  } catch (error) {
    console.error("Webhook processing error:", error);
    return new Response(
      JSON.stringify({
        error: error instanceof Error ? error.message : "Unknown error",
      }),
      { status: 500 }
    );
  }
}
```

---

## 3. Customer Portal Session

### Billing Portal

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

  try {
    // Get customer ID from Supabase
    const { data: subscription } = await supabase
      .from("subscriptions")
      .select("stripe_customer_id")
      .eq("user_id", user.id)
      .maybeSingle();

    let customerId = subscription?.stripe_customer_id;

    // Create customer if doesn't exist
    if (!customerId) {
      const customer = await stripe.customers.create({
        email: user.email,
        metadata: {
          user_id: user.id,
        },
      });
      customerId = customer.id;
    }

    // Create billing portal session
    const portalSession = await stripe.billingPortal.sessions.create({
      customer: customerId,
      return_url: `${request.nextUrl.origin}/billing`,
      configuration: process.env.STRIPE_PORTAL_CONFIG_ID,
    });

    return NextResponse.json({ url: portalSession.url });
  } catch (error) {
    console.error("Portal session error:", error);
    return NextResponse.json(
      { error: "Failed to create portal session" },
      { status: 500 }
    );
  }
}
```

---

## 4. Subscription Status Sync

### Sync to Supabase

```typescript
export async function syncSubscriptionStatus(
  subscriptionId: string,
  supabase: any
) {
  try {
    // Fetch from Stripe
    const subscription = await stripe.subscriptions.retrieve(subscriptionId);

    // Get user from metadata
    const customer = await stripe.customers.retrieve(
      subscription.customer as string
    );
    const userId = customer.metadata?.user_id;

    if (!userId) {
      console.warn("No user_id found in customer metadata");
      return;
    }

    // Sync to Supabase
    const { error } = await supabase.from("subscriptions").upsert(
      {
        user_id: userId,
        stripe_subscription_id: subscription.id,
        stripe_customer_id: subscription.customer,
        price_id: subscription.items.data[0]?.price.id,
        status: subscription.status,
        current_period_start: new Date(
          subscription.current_period_start * 1000
        ),
        current_period_end: new Date(subscription.current_period_end * 1000),
        cancel_at_period_end: subscription.cancel_at_period_end,
        canceled_at: subscription.canceled_at
          ? new Date(subscription.canceled_at * 1000)
          : null,
      },
      {
        onConflict: "stripe_subscription_id",
      }
    );

    if (error) {
      console.error("Sync error:", error);
      throw error;
    }

    return subscription;
  } catch (error) {
    console.error("Failed to sync subscription:", error);
    throw error;
  }
}

// Use in cron job or manual sync
export async function syncAllSubscriptions(supabase: any) {
  try {
    let startingAfter: string | undefined;
    let hasMore = true;

    while (hasMore) {
      const subscriptions = await stripe.subscriptions.list({
        limit: 100,
        starting_after: startingAfter,
      });

      for (const subscription of subscriptions.data) {
        await syncSubscriptionStatus(subscription.id, supabase);
      }

      hasMore = subscriptions.has_more;
      if (subscriptions.data.length > 0) {
        startingAfter = subscriptions.data[subscriptions.data.length - 1].id;
      }
    }

    console.log("Subscription sync complete");
  } catch (error) {
    console.error("Batch sync error:", error);
    throw error;
  }
}
```

---

## 5. Pricing Page Data Fetching

### Get Products and Prices

```typescript
import { stripe } from "@/lib/stripe";
import { cache } from "react";

export const getProducts = cache(async () => {
  try {
    const prices = await stripe.prices.list({
      expand: ["data.product"],
      active: true,
      type: "recurring",
    });

    const products = prices.data
      .map((price) => {
        const product = price.product as Stripe.Product;
        return {
          id: product.id,
          name: product.name,
          description: product.description,
          priceId: price.id,
          unitAmount: price.unit_amount,
          currency: price.currency,
          interval: price.recurring?.interval,
          intervalCount: price.recurring?.interval_count,
          metadata: product.metadata,
          features: (product.metadata?.features || "")
            .split(",")
            .filter(Boolean),
        };
      })
      .sort((a, b) => (a.unitAmount || 0) - (b.unitAmount || 0));

    return products;
  } catch (error) {
    console.error("Failed to fetch products:", error);
    return [];
  }
});

// Use in pricing page
export default async function PricingPage() {
  const products = await getProducts();

  return (
    <div className="grid grid-cols-3 gap-8">
      {products.map((product) => (
        <div key={product.id} className="border rounded-lg p-6">
          <h3 className="text-2xl font-bold">{product.name}</h3>
          <p className="text-gray-600">{product.description}</p>
          <div className="mt-4">
            <span className="text-4xl font-bold">
              ${(product.unitAmount! / 100).toFixed(0)}
            </span>
            <span className="text-gray-600">/{product.interval}</span>
          </div>
          <ul className="mt-6 space-y-2">
            {product.features.map((feature) => (
              <li key={feature} className="flex items-center">
                <span className="text-green-500">✓</span>
                <span className="ml-2">{feature}</span>
              </li>
            ))}
          </ul>
          <CheckoutButton priceId={product.priceId} />
        </div>
      ))}
    </div>
  );
}
```

---

## 6. Usage-Based Billing

### Record Usage

```typescript
import { stripe } from "@/lib/stripe";

export async function recordUsage({
  subscriptionItemId,
  quantity,
  timestamp = Math.floor(Date.now() / 1000),
}: {
  subscriptionItemId: string;
  quantity: number;
  timestamp?: number;
}) {
  try {
    const usageRecord = await stripe.usageRecords.create(subscriptionItemId, {
      quantity,
      timestamp,
    });

    return usageRecord;
  } catch (error) {
    console.error("Failed to record usage:", error);
    throw error;
  }
}

// Example: Record API calls
export async function recordApiCall(userId: string) {
  const supabase = createClient();

  // Get subscription
  const { data: subscription } = await supabase
    .from("subscriptions")
    .select("stripe_subscription_id")
    .eq("user_id", userId)
    .single();

  if (!subscription) return;

  // Get Stripe subscription
  const stripeSubscription = await stripe.subscriptions.retrieve(
    subscription.stripe_subscription_id
  );

  // Find metered usage item
  const meteredItem = stripeSubscription.items.data.find((item) => {
    const price = item.price as Stripe.Price;
    return price.recurring?.usage_type === "metered";
  });

  if (!meteredItem) return;

  // Record usage
  await recordUsage({
    subscriptionItemId: meteredItem.id,
    quantity: 1,
  });
}
```

### Metering in Supabase

```sql
create table public.usage_events (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.profiles on delete cascade,
  event_type text not null,
  quantity integer default 1,
  recorded_at timestamp with time zone default now(),
  synced_to_stripe boolean default false
);

-- Sync usage to Stripe periodically
create function public.sync_usage_to_stripe()
returns void as $$
declare
  event record;
begin
  for event in
    select * from public.usage_events
    where not synced_to_stripe
    and recorded_at > now() - interval '1 hour'
  loop
    -- Call recordUsage function
    perform http_post(
      'https://stripe.com/v1/subscription_items/' || event.user_id || '/usage_records',
      jsonb_build_object('quantity', event.quantity, 'timestamp', extract(epoch from event.recorded_at))
    );

    update public.usage_events
    set synced_to_stripe = true
    where id = event.id;
  end loop;
end;
$$ language plpgsql;
```

---

## Key Integration Points

- **Customer ID**: Always store `stripe_customer_id` in Supabase for bidirectional sync
- **Webhook Verification**: Always verify signature before processing
- **Idempotency**: Webhooks can fire multiple times — use `UPSERT` on subscriptions
- **Error Handling**: Webhook failures should be logged and retried
- **Testing**: Use Stripe test keys (`sk_test_*`) in development

This is production-ready. Test thoroughly with Stripe test environment first.

