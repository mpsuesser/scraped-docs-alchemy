---
url: https://alchemy.run/stripe/guides/subscriptions
title: "Sell a subscription"
description: "The full SaaS billing loop on one Cloudflare Worker — hosted Checkout, the Billing Portal, and webhooks that keep an entitlement record your app gates on."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

You have a product with a monthly price and you want people to pay for it. The shape every SaaS lands on is:

1. **Checkout** — your app creates a Stripe Customer and a hosted Checkout Session, then redirects the buyer to Stripe’s payment page.
2. **Webhooks** — Stripe tells you what happened (`paid`, `renewed`, `card declined`, `canceled`). Your app writes that to its own store.
3. **Entitlement** — the rest of your app reads that store to decide whether to serve paid features. It never calls Stripe on the request path.
4. **Billing Portal** — a customer who wants to change their card or cancel gets sent to Stripe’s hosted portal. You don’t build those screens.

Take `examples/stripe-billing`, which does exactly that on one Worker. Every snippet below is a piece of [`src/Api.ts`](https://github.com/alchemy-run/alchemy/blob/main/examples/stripe-billing/src/Api.ts); the whole file is at the end.

<svg viewBox="-4.000000000000007 28 657.2 200" style="max-width: 756px" role="img" aria-label="Your app never decides these states. Stripe moves the subscription and tells you with a webhook: checkout.session.completed, invoice.payment_failed, customer.subscription.updated or .deleted. You write the new status to your own store, which the rest of the app reads."><defs><marker id="alc-sm-arrow-a0f53o" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 160.39999999999998 63 Q 220.39999999999998 63 276.4 63" marker-end="url(#alc-sm-arrow-a0f53o)"></path><path stroke="currentColor" fill="none" d="M 312.99999999999994 86 Q 262.59999999999997 128 309.92711488161046 167.4392624013421" marker-end="url(#alc-sm-arrow-a0f53o)"></path><path stroke="currentColor" fill="none" d="M 368.2 170 Q 418.59999999999997 128 371.27288511838947 88.56073759865792" marker-end="url(#alc-sm-arrow-a0f53o)"></path><path stroke="currentColor" fill="none" d="M 400.79999999999995 63 Q 460.79999999999995 63 516.8 63" marker-end="url(#alc-sm-arrow-a0f53o)"></path><g><g><circle stroke="currentColor" fill="none" cx="13.999999999999993" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="19.999999999999993" y1="63" x2="35.99999999999999" y2="63" marker-end="url(#alc-sm-arrow-a0f53o)"></line></g><rect stroke="currentColor" fill="none" x="39.99999999999999" y="40" width="120.39999999999999" height="46" rx="23"></rect><text fill="currentColor" x="100.19999999999999" y="67" text-anchor="middle">none</text></g> <g><rect stroke="currentColor" fill="none" x="280.4" y="40" width="120.39999999999999" height="46" rx="23"></rect><text fill="currentColor" x="340.59999999999997" y="61" text-anchor="middle">active</text> <text fill="currentColor" x="340.59999999999997" y="76" text-anchor="middle">or trialing</text></g> <g><rect stroke="currentColor" fill="none" x="280.4" y="170" width="120.39999999999999" height="46" rx="23"></rect><text fill="currentColor" x="340.59999999999997" y="191" text-anchor="middle">past_due</text> <text fill="currentColor" x="340.59999999999997" y="206" text-anchor="middle">payment failed</text></g> <g><rect stroke="currentColor" fill="none" x="520.8" y="40" width="120.39999999999999" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="524.8" y="44" width="112.39999999999999" height="38" rx="19"></rect><text fill="currentColor" x="581" y="67" text-anchor="middle">canceled</text></g> <g><rect stroke="currentColor" fill="none" x="187.99999999999997" y="53" width="62.8" height="17" rx="4"></rect><text fill="currentColor" x="219.39999999999998" y="66" text-anchor="middle">checkout</text></g> <g><rect stroke="currentColor" fill="none" x="239.1317787204026" y="117.35981560033552" width="95.8" height="17" rx="4"></rect><text fill="currentColor" x="287.0317787204026" y="130.35981560033554" text-anchor="middle">payment fails</text></g> <g><rect stroke="currentColor" fill="none" x="375.96822127959734" y="118.64018439966449" width="36.4" height="17" rx="4"></rect><text fill="currentColor" x="394.16822127959733" y="131.6401843996645" text-anchor="middle">paid</text></g> <g><rect stroke="currentColor" fill="none" x="431.69999999999993" y="53" width="56.199999999999996" height="17" rx="4"></rect><text fill="currentColor" x="459.79999999999995" y="66" text-anchor="middle">deleted</text></g></svg> <svg viewBox="27.999999999999993 -4 370.59999999999997 378" style="max-width: 426px" role="img" aria-label="Your app never decides these states. Stripe moves the subscription and tells you with a webhook: checkout.session.completed, invoice.payment_failed, customer.subscription.updated or .deleted. You write the new status to your own store, which the rest of the app reads."><defs><marker id="alc-sm-arrow-n-a0f53o" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 100.19999999999999 86 Q 100.19999999999999 132 100.19999999999999 174" marker-end="url(#alc-sm-arrow-n-a0f53o)"></path><path stroke="currentColor" fill="none" d="M 112.17916666666666 178 Q 213.29999999999998 -16.151999999999987 312.5730956579854 174.45234366333207" marker-end="url(#alc-sm-arrow-n-a0f53o)"></path><path stroke="currentColor" fill="none" d="M 314.4208333333333 224 Q 213.29999999999998 418.152 114.02690434201453 227.54765633666793" marker-end="url(#alc-sm-arrow-n-a0f53o)"></path><path stroke="currentColor" fill="none" d="M 100.19999999999999 224 Q 100.19999999999999 270 100.19999999999999 312" marker-end="url(#alc-sm-arrow-n-a0f53o)"></path><g><g><circle stroke="currentColor" fill="none" cx="100.19999999999999" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="100.19999999999999" y1="20" x2="100.19999999999999" y2="36" marker-end="url(#alc-sm-arrow-n-a0f53o)"></line></g><rect stroke="currentColor" fill="none" x="39.99999999999999" y="40" width="120.39999999999999" height="46" rx="23"></rect><text fill="currentColor" x="100.19999999999999" y="67" text-anchor="middle">none</text></g> <g><rect stroke="currentColor" fill="none" x="39.99999999999999" y="178" width="120.39999999999999" height="46" rx="23"></rect><text fill="currentColor" x="100.19999999999999" y="199" text-anchor="middle">active</text> <text fill="currentColor" x="100.19999999999999" y="214" text-anchor="middle">or trialing</text></g> <g><rect stroke="currentColor" fill="none" x="266.2" y="178" width="120.39999999999999" height="46" rx="23"></rect><text fill="currentColor" x="326.4" y="199" text-anchor="middle">past_due</text> <text fill="currentColor" x="326.4" y="214" text-anchor="middle">payment failed</text></g> <g><rect stroke="currentColor" fill="none" x="39.99999999999999" y="316" width="120.39999999999999" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="43.99999999999999" y="320" width="112.39999999999999" height="38" rx="19"></rect><text fill="currentColor" x="100.19999999999999" y="343" text-anchor="middle">canceled</text></g> <g><rect stroke="currentColor" fill="none" x="68.79999999999998" y="121" width="62.8" height="17" rx="4"></rect><text fill="currentColor" x="100.19999999999999" y="134" text-anchor="middle">checkout</text></g> <g><rect stroke="currentColor" fill="none" x="117.038065581163" y="70.03708591583302" width="95.8" height="17" rx="4"></rect><text fill="currentColor" x="207.838065581163" y="83.03708591583302" text-anchor="end">payment fails</text></g> <g><rect stroke="currentColor" fill="none" x="213.76193441883694" y="311.962914084167" width="36.4" height="17" rx="4"></rect><text fill="currentColor" x="218.76193441883694" y="324.962914084167" text-anchor="start">paid</text></g> <g><rect stroke="currentColor" fill="none" x="72.1" y="259" width="56.199999999999996" height="17" rx="4"></rect><text fill="currentColor" x="100.19999999999999" y="272" text-anchor="middle">deleted</text></g></svg>

Your app never decides these states. Stripe moves the subscription and tells you with a webhook: checkout.session.completed, invoice.payment\_failed, customer.subscription.updated or.deleted. You write the new status to your own store, which the rest of the app reads.

## The catalog lives on the Worker

The Product, its Price, a launch Coupon, and the Billing Portal configuration are Stack resources declared *inside* the Worker’s Effect. They deploy with the Worker and are destroyed with it.

```typescript
export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const product = yield* Stripe.Product("Pro", {
      name: "Pro",
      description: "Billed monthly",
    });
    const price = yield* Stripe.Price("ProMonthly", {
      product,
      currency: "usd",
      unitAmount: 2000,
      recurring: { interval: "month" },
    });
    yield* Stripe.Coupon("Launch20", {
      percentOff: 20,
      duration: "once",
      name: "Launch 20%",
    });

    const portalConfig = yield* Stripe.BillingPortalConfiguration("Portal", {
      name: "Pro customer portal",
      features: {
        invoiceHistory: { enabled: true },
        paymentMethodUpdate: { enabled: true },
        customerUpdate: { enabled: true, allowedUpdates: ["email"] },
        subscriptionCancel: { enabled: true, mode: "at_period_end" },
      },
    });
    // ...
```

`Price` takes the `Product` directly — no id plumbing.

## Carry ids from plan time to runtime

`price.id` is an `Output` — it doesn’t have a value until the Price is created. Yielding an `Output` inside the Worker’s Effect gives you an accessor you can yield again from a request handler:

```typescript
const priceId = yield* price.id;
const portalConfigId = yield* portalConfig.id;
```

That’s the whole mechanism for getting a resource’s id into a route. There is no environment variable to name and no string to interpolate.

## Bind the Stripe calls the routes make

Three HTTP bindings, one per Stripe call the Worker will make at runtime. Each one adds its permission (`customers_write`, `checkout_sessions_write`, `billing_portal_write`) to the Worker’s Stripe token and returns a callable:

```typescript
const createCustomer = yield* Stripe.CreateCustomer();
const createCheckout = yield* Stripe.CreateCheckoutSession();
const createPortal = yield* Stripe.CreateBillingPortalSession();
```

And the store the webhook handler writes to:

```typescript
const entitlements = yield* Cloudflare.KV.Namespace("Entitlements");
const kv = yield* Cloudflare.KV.ReadWriteNamespace(entitlements);
```

`consumeEvents` provisions the `WebhookEndpoint` pointed at this Worker, enables exactly the events you list, and binds the signing secret. The handler runs once per verified delivery with a typed `event`.

```typescript
yield* Stripe.consumeEvents(
  "Events",
  {
    events: [
      Stripe.CheckoutSessionCompleted,
      Stripe.CustomerSubscriptionCreated,
      Stripe.CustomerSubscriptionUpdated,
      Stripe.CustomerSubscriptionDeleted,
      Stripe.InvoicePaymentFailed,
    ],
  },
  Effect.fn(function* (event) {
    const write = (entitlement: Entitlement) =>
      kv
        .put(entitlement.customerId, JSON.stringify(entitlement))
        .pipe(Effect.orDie);

    switch (event.type) {
      case "checkout.session.completed": {
        const session = event.object;
        const customerId = idOf(session.customer);
        if (customerId === null) return;
        yield* write({
          customerId,
          status: "active",
          priceId: yield* priceId,
          subscriptionId: idOf(session.subscription),
          updatedAt: Date.now(),
        });
        return;
      }
      case "customer.subscription.created":
      case "customer.subscription.updated": {
        const subscription = event.object;
        yield* write({
          customerId: idOf(subscription.customer)!,
          status:
            subscription.status === "active" ||
            subscription.status === "trialing"
              ? "active"
              : subscription.status === "past_due"
                ? "past_due"
                : "canceled",
          priceId: subscription.items.data[0]?.price.id ?? null,
          subscriptionId: subscription.id,
          updatedAt: Date.now(),
        });
        return;
      }
      case "customer.subscription.deleted": {
        // ... status: "canceled"
      }
      case "invoice.payment_failed": {
        // ... status: "past_due" — Stripe retries; nudge to the portal
      }
    }
  }),
);
```

The `Entitlement` record is deliberately small — customer, status, which price, which subscription. It’s the only thing the rest of your app needs.

## The routes

Everything above is plan time. The `fetch` handler is what runs per request.

### POST /checkout — start a subscription

Create the Customer first so every webhook that follows carries a stable `cus_…` id you can key on, then create the Checkout Session for the deployed Price and hand back the hosted URL:

```typescript
if (request.method === "POST" && url.pathname === "/checkout") {
  const body = (yield* request.json) as { email?: string };
  if (!body.email) {
    return yield* HttpServerResponse.json(
      { error: "email is required" },
      { status: 400 },
    );
  }

  const customer = yield* createCustomer({ email: body.email }).pipe(
    Effect.orDie,
  );
  const session = yield* createCheckout({
    mode: "subscription",
    customer: customer.id,
    line_items: [{ price: yield* priceId, quantity: 1 }],
    allow_promotion_codes: true,
    success_url: \`${url.origin}/welcome?session_id={CHECKOUT_SESSION_ID}\`,
    cancel_url: \`${url.origin}/pricing\`,
  }).pipe(Effect.orDie);

  return yield* HttpServerResponse.json(
    { customerId: customer.id, checkoutUrl: session.url },
    { status: 201 },
  );
}
```

Your frontend redirects to `checkoutUrl`. Stripe collects the card and sends the buyer to `success_url`.

### POST /portal — let them manage it

```typescript
if (request.method === "POST" && url.pathname === "/portal") {
  const body = (yield* request.json) as { customerId?: string };
  // ...
  const session = yield* createPortal({
    customer: body.customerId,
    configuration: yield* portalConfigId,
    return_url: \`${url.origin}/account\`,
  }).pipe(Effect.orDie);
  return yield* HttpServerResponse.json({ portalUrl: session.url });
}
```

The portal honours the `BillingPortalConfiguration` declared above: invoice history, card update, email update, cancel at period end.

### GET /subscription/:customerId — what to gate on

```typescript
if (
  request.method === "GET" &&
  url.pathname.startsWith("/subscription/")
) {
  const customerId = url.pathname.slice("/subscription/".length);
  const entitlement = yield* kv
    .get<Entitlement>(customerId, "json")
    .pipe(Effect.orDie);
  return yield* HttpServerResponse.json(
    entitlement ?? { customerId, status: "none" },
  );
}
```

This is a KV read. It’s what your app calls on every authenticated request to decide what to show.

## Provide the layers

The Worker Effect closes by providing every binding’s implementation in one `Effect.provide([...])`:

```typescript
}).pipe(
    Effect.provide([
      Cloudflare.KV.ReadWriteNamespaceBinding,
      Stripe.CreateCustomerHttp,
      Stripe.CreateCheckoutSessionHttp,
      Stripe.CreateBillingPortalSessionHttp,
      Stripe.ConsumeEventsLive,
    ]),
  ),
) {}
```

## Try it

```sh
cd examples/stripe-billing
pnpm alchemy deploy
```

```sh
curl -X POST "$URL/checkout" -H 'content-type: application/json' \
  -d '{"email":"you@example.com"}'
# → { "customerId": "cus_…", "checkoutUrl": "https://checkout.stripe.com/…" }
```

Open `checkoutUrl` in a browser and pay with Stripe’s test card `4242 4242 4242 4242`. Within a few seconds:

```sh
curl "$URL/subscription/cus_…"
# → { "customerId": "cus_…", "status": "active", "priceId": "price_…", … }
```

Cancel from the portal (`POST /portal` → open `portalUrl`) and the same call returns `"status": "canceled"`.

The integration test in `test/integ.test.ts` runs this loop without a browser by attaching Stripe’s `tok_visa` test card and creating the subscription directly, then asserting the webhook-driven entitlement reaches `active` and then `canceled`.

## Full file

`examples/stripe-billing/src/Api.ts`

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Stripe from "@alchemy.run/stripe";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

interface Entitlement {
  customerId: string;
  status: "active" | "past_due" | "canceled";
  priceId: string | null;
  subscriptionId: string | null;
  updatedAt: number;
}

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  {
    main: import.meta.url,
  },
  Effect.gen(function* () {
    const product = yield* Stripe.Product("Pro", {
      name: "Pro",
      description: "Billed monthly",
    });
    const price = yield* Stripe.Price("ProMonthly", {
      product,
      currency: "usd",
      unitAmount: 2000,
      recurring: { interval: "month" },
    });
    yield* Stripe.Coupon("Launch20", {
      percentOff: 20,
      duration: "once",
      name: "Launch 20%",
    });

    const portalConfig = yield* Stripe.BillingPortalConfiguration("Portal", {
      name: "Pro customer portal",
      features: {
        invoiceHistory: { enabled: true },
        paymentMethodUpdate: { enabled: true },
        customerUpdate: { enabled: true, allowedUpdates: ["email"] },
        subscriptionCancel: { enabled: true, mode: "at_period_end" },
      },
    });

    const priceId = yield* price.id;
    const portalConfigId = yield* portalConfig.id;

    const createCustomer = yield* Stripe.CreateCustomer();
    const createCheckout = yield* Stripe.CreateCheckoutSession();
    const createPortal = yield* Stripe.CreateBillingPortalSession();

    const entitlements = yield* Cloudflare.KV.Namespace("Entitlements");
    const kv = yield* Cloudflare.KV.ReadWriteNamespace(entitlements);

    yield* Stripe.consumeEvents(
      "Events",
      {
        events: [
          Stripe.CheckoutSessionCompleted,
          Stripe.CustomerSubscriptionCreated,
          Stripe.CustomerSubscriptionUpdated,
          Stripe.CustomerSubscriptionDeleted,
          Stripe.InvoicePaymentFailed,
        ],
      },
      Effect.fn(function* (event) {
        const write = (entitlement: Entitlement) =>
          kv
            .put(entitlement.customerId, JSON.stringify(entitlement))
            .pipe(Effect.orDie);

        switch (event.type) {
          case "checkout.session.completed": {
            const session = event.object;
            const customerId = idOf(session.customer);
            if (customerId === null) return;
            yield* write({
              customerId,
              status: "active",
              priceId: yield* priceId,
              subscriptionId: idOf(session.subscription),
              updatedAt: Date.now(),
            });
            return;
          }
          case "customer.subscription.created":
          case "customer.subscription.updated": {
            const subscription = event.object;
            yield* write({
              customerId: idOf(subscription.customer)!,
              status:
                subscription.status === "active" ||
                subscription.status === "trialing"
                  ? "active"
                  : subscription.status === "past_due"
                    ? "past_due"
                    : "canceled",
              priceId: subscription.items.data[0]?.price.id ?? null,
              subscriptionId: subscription.id,
              updatedAt: Date.now(),
            });
            return;
          }
          case "customer.subscription.deleted": {
            const subscription = event.object;
            yield* write({
              customerId: idOf(subscription.customer)!,
              status: "canceled",
              priceId: null,
              subscriptionId: subscription.id,
              updatedAt: Date.now(),
            });
            return;
          }
          case "invoice.payment_failed": {
            const invoice = event.object;
            const customerId = idOf(invoice.customer);
            if (customerId === null) return;
            const current = yield* kv
              .get<Entitlement>(customerId, "json")
              .pipe(Effect.orDie);
            yield* write({
              customerId,
              status: "past_due",
              priceId: current?.priceId ?? null,
              subscriptionId: current?.subscriptionId ?? null,
              updatedAt: Date.now(),
            });
            return;
          }
        }
      }),
    );

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const url = new URL(request.originalUrl);

        if (request.method === "POST" && url.pathname === "/checkout") {
          const body = (yield* request.json) as { email?: string };
          if (!body.email) {
            return yield* HttpServerResponse.json(
              { error: "email is required" },
              { status: 400 },
            );
          }

          const customer = yield* createCustomer({ email: body.email }).pipe(
            Effect.orDie,
          );
          const session = yield* createCheckout({
            mode: "subscription",
            customer: customer.id,
            line_items: [{ price: yield* priceId, quantity: 1 }],
            allow_promotion_codes: true,
            success_url: \`${url.origin}/welcome?session_id={CHECKOUT_SESSION_ID}\`,
            cancel_url: \`${url.origin}/pricing\`,
          }).pipe(Effect.orDie);

          return yield* HttpServerResponse.json(
            { customerId: customer.id, checkoutUrl: session.url },
            { status: 201 },
          );
        }

        if (request.method === "POST" && url.pathname === "/portal") {
          const body = (yield* request.json) as { customerId?: string };
          if (!body.customerId) {
            return yield* HttpServerResponse.json(
              { error: "customerId is required" },
              { status: 400 },
            );
          }
          const session = yield* createPortal({
            customer: body.customerId,
            configuration: yield* portalConfigId,
            return_url: \`${url.origin}/account\`,
          }).pipe(Effect.orDie);
          return yield* HttpServerResponse.json({ portalUrl: session.url });
        }

        if (
          request.method === "GET" &&
          url.pathname.startsWith("/subscription/")
        ) {
          const customerId = url.pathname.slice("/subscription/".length);
          const entitlement = yield* kv
            .get<Entitlement>(customerId, "json")
            .pipe(Effect.orDie);
          return yield* HttpServerResponse.json(
            entitlement ?? { customerId, status: "none" },
          );
        }

        if (url.pathname === "/welcome" || url.pathname === "/pricing") {
          return HttpServerResponse.text(
            url.pathname === "/welcome"
              ? "Thanks — your subscription is active."
              : "Pricing page.",
          );
        }

        return yield* HttpServerResponse.json(
          { error: "Not found" },
          { status: 404 },
        );
      }),
    };
  }).pipe(
    Effect.provide([
      Cloudflare.KV.ReadWriteNamespaceBinding,
      Stripe.CreateCustomerHttp,
      Stripe.CreateCheckoutSessionHttp,
      Stripe.CreateBillingPortalSessionHttp,
      Stripe.ConsumeEventsLive,
    ]),
  ),
) {}

const idOf = (
  ref: string | { id: string } | { id?: string } | null | undefined,
): string | null =>
  ref == null ? null : typeof ref === "string" ? ref : (ref.id ?? null);
```

## Next

- [Connect](connect.md) — onboard other businesses and pay them out.
- [BillingPortalConfiguration](https://alchemy.run/providers/stripe/reference/billing#billingportalconfiguration), [Price](https://alchemy.run/providers/stripe/reference/product#price), [WebhookEndpoint](https://alchemy.run/providers/stripe/reference/webhook#webhookendpoint) reference.
