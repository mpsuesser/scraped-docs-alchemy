---
url: https://alchemy.run/stripe
title: "Stripe"
description: "Sell subscriptions and onboard merchants from a Cloudflare Worker — catalog, Checkout, the Billing Portal, Connect, and webhooks declared next to the code that uses them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Stripe is financial infrastructure for the internet: payments, subscriptions and billing, invoicing, tax, fraud prevention and payouts to other businesses.

It keeps what you sell, products and their prices, who buys it, and the record of every payment and subscription. Its hosted pages, Checkout and the Billing Portal, collect card details so you never touch them, and webhooks tell your app what happened.

**New to Stripe?** Start with [What is Stripe?](stripe/what-is-stripe.md).

## A quick example

```typescript
export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    // Catalog — deployed with the Worker.
    const product = yield* Stripe.Product("Pro", { name: "Pro" });
    const price = yield* Stripe.Price("ProMonthly", {
      product,
      currency: "usd",
      unitAmount: 2000,
      recurring: { interval: "month" },
    });
    const priceId = yield* price.id;

    // Runtime calls — each adds its permission to the Worker's token.
    const createCustomer = yield* Stripe.CreateCustomer();
    const createCheckout = yield* Stripe.CreateCheckoutSession();

    // Stripe → app.
    yield* Stripe.consumeEvents(
      "Events",
      { events: [Stripe.CustomerSubscriptionCreated] },
      Effect.fn(function* (event) {
        yield* Effect.log(\`${event.object.customer} subscribed\`);
      }),
    );

    return {
      fetch: Effect.gen(function* () {
        const customer = yield* createCustomer({ email: "a@example.com" });
        const session = yield* createCheckout({
          mode: "subscription",
          customer: customer.id,
          line_items: [{ price: yield* priceId, quantity: 1 }],
          success_url: "https://example.com/welcome",
          cancel_url: "https://example.com/pricing",
        });
        return HttpServerResponse.redirect(session.url!);
      }).pipe(Effect.orDie),
    };
  }).pipe(
    Effect.provide([
      Stripe.CreateCustomerHttp,
      Stripe.CreateCheckoutSessionHttp,
      Stripe.ConsumeEventsLive,
    ]),
  ),
) {}
```

Resources like `Product`, `Price`, `Coupon` and `WebhookEndpoint` are managed by the Stack: Alchemy creates, updates and deletes them, and deactivates the ones Stripe won’t delete. Bindings like `CreateCustomer` and `CreateCheckoutSession` are what the Worker calls while it runs. Each one adds the permission it needs to a [`RestrictedApiKey`](stripe/accounts-and-keys.md), and you provide its `*Http` Layer once. And an output like `price.id` is yielded in Construction to get an accessor, then yielded again inside a handler, which is how a resource’s id reaches the running Worker.

## Where to start

**Want to build?** Connect your account in [Setup](stripe/setup.md), then [Sell a subscription](stripe/guides/subscriptions.md) builds Checkout, the Billing Portal and webhooks in one Worker. Paying other businesses? [Onboard merchants with Connect](stripe/guides/connect.md).

**Looking for something specific?** Every section of these docs is listed below, and every resource is in the generated [API reference](https://alchemy.run/providers).

## The docs

### Accounts & access

- [Accounts and API keys](stripe/accounts-and-keys.md)
