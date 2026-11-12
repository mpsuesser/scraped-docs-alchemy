---
url: https://alchemy.run/stripe/guides/connect
title: "Onboard merchants with Connect"
description: "Build a Connect platform on a Cloudflare Worker — create Express accounts, send merchants through hosted onboarding, and learn from account.updated when they can take payments."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Stripe Connect is for platforms: a marketplace, a SaaS that invoices on behalf of its customers, anything where *other businesses* get paid through your app. Each of those businesses becomes a **connected account** with its own Stripe identity. Your platform creates it, sends the owner through Stripe’s onboarding form, and — once Stripe has verified them — can route payments to it.

That is a state machine, not a form submission. The parts every platform needs:

1. **Create the account** when a merchant signs up. Store its `acct_…` id next to your own user record.
2. **Send them to hosted onboarding** via an Account Link. Links are single-use and expire in minutes; merchants abandon them and come back tomorrow. You must be able to mint a new one on demand.
3. **Learn when they’re done.** Stripe sets `charges_enabled` on the account after verification and fires `account.updated`. That webhook — not the merchant returning to your `return_url` — is the source of truth.
4. **Gate on it.** Your app checks your stored `charges_enabled` before it lets the merchant sell.

Take `examples/stripe-connect`, which implements all four on one Worker with a D1 table. Every snippet is a piece of [`src/Api.ts`](https://github.com/alchemy-run/alchemy/blob/main/examples/stripe-connect/src/Api.ts); the full file is at the end.

<svg viewBox="-4 28 788 200" style="max-width: 906px" role="img" aria-label="Your platform never sets these states. Stripe moves the account along and tells you with account.updated, and your app gates payments on the charges_enabled it stored."><defs><marker id="alc-sm-arrow-7bzdth" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 204 63 Q 264 63 320 63" marker-end="url(#alc-sm-arrow-7bzdth)"></path><path stroke="currentColor" fill="none" d="M 488 63 Q 548 63 604 63" marker-end="url(#alc-sm-arrow-7bzdth)"></path><path stroke="currentColor" fill="none" d="M 690 86 Q 690 128 690 166" marker-end="url(#alc-sm-arrow-7bzdth)"></path><g><g><circle stroke="currentColor" fill="none" cx="14" cy="63" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="20" y1="63" x2="36" y2="63" marker-end="url(#alc-sm-arrow-7bzdth)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="61" text-anchor="middle">created</text> <text fill="currentColor" x="122" y="76" text-anchor="middle">connected account</text></g> <g><rect stroke="currentColor" fill="none" x="324" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="406" y="61" text-anchor="middle">onboarding</text> <text fill="currentColor" x="406" y="76" text-anchor="middle">on Stripe's form</text></g> <g><rect stroke="currentColor" fill="none" x="608" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="690" y="61" text-anchor="middle">details submitted</text> <text fill="currentColor" x="690" y="76" text-anchor="middle">Stripe verifying</text></g> <g><rect stroke="currentColor" fill="none" x="608" y="170" width="164" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="612" y="174" width="156" height="38" rx="19"></rect><text fill="currentColor" x="690" y="191" text-anchor="middle">charges_enabled</text> <text fill="currentColor" x="690" y="206" text-anchor="middle">can take payments</text></g> <g><rect stroke="currentColor" fill="none" x="208.5" y="53" width="109" height="17" rx="4"></rect><text fill="currentColor" x="263" y="66" text-anchor="middle">onboarding link</text></g> <g><rect stroke="currentColor" fill="none" x="485.9" y="53" width="122.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="547" y="66" text-anchor="middle">merchant finishes</text></g> <g><rect stroke="currentColor" fill="none" x="635.5" y="117" width="109" height="17" rx="4"></rect><text fill="currentColor" x="690" y="130" text-anchor="middle">account.updated</text></g></svg> <svg viewBox="28 -4 471 378" style="max-width: 542px" role="img" aria-label="Your platform never sets these states. Stripe moves the account along and tells you with account.updated, and your app gates payments on the charges_enabled it stored."><defs><marker id="alc-sm-arrow-n-7bzdth" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><path stroke="currentColor" fill="none" d="M 122 86 Q 122 132 122 174" marker-end="url(#alc-sm-arrow-n-7bzdth)"></path><path stroke="currentColor" fill="none" d="M 122 224 Q 122 270 122 312" marker-end="url(#alc-sm-arrow-n-7bzdth)"></path><path stroke="currentColor" fill="none" d="M 204 339 Q 263.5 339 319 339" marker-end="url(#alc-sm-arrow-n-7bzdth)"></path><g><g><circle stroke="currentColor" fill="none" cx="122" cy="14" r="6"></circle><line stroke-opacity="0.2" stroke="currentColor" x1="122" y1="20" x2="122" y2="36" marker-end="url(#alc-sm-arrow-n-7bzdth)"></line></g><rect stroke="currentColor" fill="none" x="40" y="40" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="61" text-anchor="middle">created</text> <text fill="currentColor" x="122" y="76" text-anchor="middle">connected account</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="178" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="199" text-anchor="middle">onboarding</text> <text fill="currentColor" x="122" y="214" text-anchor="middle">on Stripe's form</text></g> <g><rect stroke="currentColor" fill="none" x="40" y="316" width="164" height="46" rx="23"></rect><text fill="currentColor" x="122" y="337" text-anchor="middle">details submitted</text> <text fill="currentColor" x="122" y="352" text-anchor="middle">Stripe verifying</text></g> <g><rect stroke="currentColor" fill="none" x="323" y="316" width="164" height="46" rx="23"></rect><rect stroke="currentColor" fill="none" x="327" y="320" width="156" height="38" rx="19"></rect><text fill="currentColor" x="405" y="337" text-anchor="middle">charges_enabled</text> <text fill="currentColor" x="405" y="352" text-anchor="middle">can take payments</text></g> <g><rect stroke="currentColor" fill="none" x="67.5" y="121" width="109" height="17" rx="4"></rect><text fill="currentColor" x="122" y="134" text-anchor="middle">onboarding link</text></g> <g><rect stroke="currentColor" fill="none" x="60.900000000000006" y="259" width="122.19999999999999" height="17" rx="4"></rect><text fill="currentColor" x="122" y="272" text-anchor="middle">merchant finishes</text></g> <g><rect stroke="currentColor" fill="none" x="208" y="329" width="109" height="17" rx="4"></rect><text fill="currentColor" x="262.5" y="342" text-anchor="middle">account.updated</text></g></svg>

Your platform never sets these states. Stripe moves the account along and tells you with account.updated, and your app gates payments on the charges\_enabled it stored.

## The merchant table

One row per connected account, keyed by Stripe’s id. The three capability flags mirror what Stripe reports; `updated_at` moves when a webhook lands.

```sql
-- migrations/0001_init.sql
CREATE TABLE merchants (
  id text PRIMARY KEY,
  email text NOT NULL,
  details_submitted integer NOT NULL DEFAULT 0,
  charges_enabled integer NOT NULL DEFAULT 0,
  payouts_enabled integer NOT NULL DEFAULT 0,
  created_at integer NOT NULL DEFAULT (unixepoch()),
  updated_at integer NOT NULL DEFAULT (unixepoch())
);
```

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Database = Cloudflare.D1.Database("Database", {
  migrations: "./migrations",
});
```

## Bind the database and the Stripe calls

```typescript
export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  { main: import.meta.url },
  Effect.gen(function* () {
    const db = yield* Cloudflare.D1.QueryDatabase(Database);
    const createAccount = yield* Stripe.CreateAccount();
    const createAccountLink = yield* Stripe.CreateAccountLink();
    // ...
```

Two Stripe bindings: one to create accounts, one to mint onboarding links. Both add `accounts_write` to the Worker’s token.

## Listen for account.updated

This is the piece most first attempts skip. Stripe fires `account.updated` on *every* change to *any* connected account — most importantly when the merchant finishes onboarding and `charges_enabled` flips to `true`. `consumeEvents` provisions the endpoint; the handler copies the flags into D1.

```typescript
yield* Stripe.consumeEvents(
  "Events",
  { events: [Stripe.AccountUpdated] },
  Effect.fn(function* (event) {
    const account = event.object;
    yield* db
      .prepare(
        \`UPDATE merchants
           SET details_submitted = ?, charges_enabled = ?, payouts_enabled = ?, updated_at = unixepoch()
         WHERE id = ?\`,
      )
      .bind(
        account.details_submitted ? 1 : 0,
        account.charges_enabled ? 1 : 0,
        account.payouts_enabled ? 1 : 0,
        account.id,
      )
      .run()
      .pipe(Effect.orDie);
  }),
);
```

## Minting an onboarding link

Account Links need two URLs. `return_url` is where the merchant lands when they finish. `refresh_url` is where Stripe sends them if the link has expired — your job there is to mint a fresh one. Factor it out because two routes need it:

```typescript
const onboardingLink = (merchantId: string, origin: string) =>
  createAccountLink({
    account: merchantId,
    type: "account_onboarding",
    return_url: \`${origin}/merchants/${merchantId}/onboarded\`,
    refresh_url: \`${origin}/merchants/${merchantId}/onboarding/refresh\`,
  });
```

## The routes

### POST /merchants — sign a merchant up

Create the Express account requesting the capabilities the platform needs, store the id, and return the first onboarding link:

```typescript
if (request.method === "POST" && url.pathname === "/merchants") {
  const body = (yield* request.json) as { email?: string };
  if (!body.email) {
    return yield* HttpServerResponse.json(
      { error: "email is required" },
      { status: 400 },
    );
  }

  const account = yield* createAccount({
    type: "express",
    country: "US",
    email: body.email,
    capabilities: {
      card_payments: { requested: true },
      transfers: { requested: true },
    },
  }).pipe(Effect.orDie);

  yield* db
    .prepare("INSERT INTO merchants (id, email) VALUES (?, ?)")
    .bind(account.id, body.email)
    .run()
    .pipe(Effect.orDie);

  const link = yield* onboardingLink(account.id, url.origin).pipe(
    Effect.orDie,
  );

  return yield* HttpServerResponse.json(
    { merchantId: account.id, onboardingUrl: link.url },
    { status: 201 },
  );
}
```

`type: "express"` gives merchants a lightweight Stripe dashboard and puts Stripe in charge of the onboarding UI. `standard` and `custom` trade that off differently — see [Stripe’s account types](https://docs.stripe.com/connect/accounts).

### POST /merchants/:id/onboarding — resume

A merchant who abandoned onboarding comes back. Check they exist, mint a new link:

```typescript
if (
  request.method === "POST" &&
  segments.length === 3 &&
  segments[0] === "merchants" &&
  segments[2] === "onboarding"
) {
  const merchantId = segments[1];
  const row = yield* db
    .prepare("SELECT id FROM merchants WHERE id = ?")
    .bind(merchantId)
    .first<{ id: string }>()
    .pipe(Effect.orDie);
  if (row === null) {
    return yield* HttpServerResponse.json(
      { error: "Unknown merchant" },
      { status: 404 },
    );
  }
  const link = yield* onboardingLink(merchantId, url.origin).pipe(
    Effect.orDie,
  );
  return yield* HttpServerResponse.json({ onboardingUrl: link.url });
}
```

Your `refresh_url` page should call this and redirect.

### GET /merchants/:id — are they live?

```typescript
if (
  request.method === "GET" &&
  segments.length === 2 &&
  segments[0] === "merchants"
) {
  const merchant = yield* db
    .prepare("SELECT * FROM merchants WHERE id = ?")
    .bind(segments[1])
    .first<Merchant>()
    .pipe(Effect.orDie);
  if (merchant === null) {
    return yield* HttpServerResponse.json(
      { error: "Unknown merchant" },
      { status: 404 },
    );
  }
  return yield* HttpServerResponse.json({
    id: merchant.id,
    email: merchant.email,
    onboarded: merchant.details_submitted === 1,
    chargesEnabled: merchant.charges_enabled === 1,
    payoutsEnabled: merchant.payouts_enabled === 1,
    updatedAt: merchant.updated_at,
  });
}
```

`chargesEnabled` is the gate. Until it’s `true`, keep the merchant on the onboarding path.

## Provide the layers

```typescript
}).pipe(
    Effect.provide([
      Cloudflare.D1.QueryDatabaseBinding,
      Stripe.CreateAccountHttp,
      Stripe.CreateAccountLinkHttp,
      Stripe.ConsumeEventsLive,
    ]),
  ),
) {}
```

## Try it

```sh
cd examples/stripe-connect
pnpm alchemy deploy
```

```sh
curl -X POST "$URL/merchants" -H 'content-type: application/json' \
  -d '{"email":"merchant@example.com"}'
# → { "merchantId": "acct_…", "onboardingUrl": "https://connect.stripe.com/…" }
```

Open `onboardingUrl` and complete the test-mode form (Stripe pre-fills most of it). Then:

```sh
curl "$URL/merchants/acct_…"
# → { "onboarded": true, "chargesEnabled": true, … }
```

Or skip the form: `POST /merchants/acct_…/onboarding` proves link minting, and any change to the account — even a metadata edit in the Dashboard — fires `account.updated` and bumps `updatedAt`. That’s what `test/integ.test.ts` does.

## What’s next on a real platform

This example stops at “the merchant can take payments”. From here a platform typically:

- creates Checkout Sessions with `payment_intent_data.transfer_data.destination` set to the merchant’s `acct_…` so funds route to them, or
- creates Transfers from its own balance after the fact, and
- listens for `payout.*` events on connected accounts.

Those are ordinary `Create*` bindings and `consumeEvents` classes in the same shape as above.

## Full file

`examples/stripe-connect/src/Api.ts`

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Stripe from "@alchemy.run/stripe";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { HttpServerRequest } from "effect/http/HttpServerRequest";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Database } from "./database.ts";

interface Merchant {
  id: string;
  email: string;
  details_submitted: number;
  charges_enabled: number;
  payouts_enabled: number;
  created_at: number;
  updated_at: number;
}

export default class Api extends Cloudflare.Worker<Api>()(
  "Api",
  {
    main: import.meta.url,
  },
  Effect.gen(function* () {
    const db = yield* Cloudflare.D1.QueryDatabase(Database);
    const createAccount = yield* Stripe.CreateAccount();
    const createAccountLink = yield* Stripe.CreateAccountLink();

    yield* Stripe.consumeEvents(
      "Events",
      { events: [Stripe.AccountUpdated] },
      Effect.fn(function* (event) {
        const account = event.object;
        yield* db
          .prepare(
            \`UPDATE merchants
               SET details_submitted = ?, charges_enabled = ?, payouts_enabled = ?, updated_at = unixepoch()
             WHERE id = ?\`,
          )
          .bind(
            account.details_submitted ? 1 : 0,
            account.charges_enabled ? 1 : 0,
            account.payouts_enabled ? 1 : 0,
            account.id,
          )
          .run()
          .pipe(Effect.orDie);
      }),
    );

    const onboardingLink = (merchantId: string, origin: string) =>
      createAccountLink({
        account: merchantId,
        type: "account_onboarding",
        return_url: \`${origin}/merchants/${merchantId}/onboarded\`,
        refresh_url: \`${origin}/merchants/${merchantId}/onboarding/refresh\`,
      });

    return {
      fetch: Effect.gen(function* () {
        const request = yield* HttpServerRequest;
        const url = new URL(request.originalUrl);
        const segments = url.pathname.split("/").filter(Boolean);

        if (request.method === "GET" && url.pathname === "/") {
          return HttpServerResponse.text("ok");
        }

        if (request.method === "POST" && url.pathname === "/merchants") {
          const body = (yield* request.json) as { email?: string };
          if (!body.email) {
            return yield* HttpServerResponse.json(
              { error: "email is required" },
              { status: 400 },
            );
          }

          const account = yield* createAccount({
            type: "express",
            country: "US",
            email: body.email,
            capabilities: {
              card_payments: { requested: true },
              transfers: { requested: true },
            },
          }).pipe(Effect.orDie);

          yield* db
            .prepare("INSERT INTO merchants (id, email) VALUES (?, ?)")
            .bind(account.id, body.email)
            .run()
            .pipe(Effect.orDie);

          const link = yield* onboardingLink(account.id, url.origin).pipe(
            Effect.orDie,
          );

          return yield* HttpServerResponse.json(
            { merchantId: account.id, onboardingUrl: link.url },
            { status: 201 },
          );
        }

        if (
          request.method === "POST" &&
          segments.length === 3 &&
          segments[0] === "merchants" &&
          segments[2] === "onboarding"
        ) {
          const merchantId = segments[1];
          const row = yield* db
            .prepare("SELECT id FROM merchants WHERE id = ?")
            .bind(merchantId)
            .first<{ id: string }>()
            .pipe(Effect.orDie);
          if (row === null) {
            return yield* HttpServerResponse.json(
              { error: "Unknown merchant" },
              { status: 404 },
            );
          }
          const link = yield* onboardingLink(merchantId, url.origin).pipe(
            Effect.orDie,
          );
          return yield* HttpServerResponse.json({ onboardingUrl: link.url });
        }

        if (
          request.method === "GET" &&
          segments.length === 3 &&
          segments[0] === "merchants" &&
          (segments[2] === "onboarded" || segments[2] === "refresh")
        ) {
          return HttpServerResponse.text(
            segments[2] === "onboarded"
              ? "Onboarding complete. You can close this tab."
              : "That onboarding link expired. Request a new one from the platform.",
          );
        }

        if (
          request.method === "GET" &&
          segments.length === 2 &&
          segments[0] === "merchants"
        ) {
          const merchant = yield* db
            .prepare("SELECT * FROM merchants WHERE id = ?")
            .bind(segments[1])
            .first<Merchant>()
            .pipe(Effect.orDie);
          if (merchant === null) {
            return yield* HttpServerResponse.json(
              { error: "Unknown merchant" },
              { status: 404 },
            );
          }
          return yield* HttpServerResponse.json({
            id: merchant.id,
            email: merchant.email,
            onboarded: merchant.details_submitted === 1,
            chargesEnabled: merchant.charges_enabled === 1,
            payoutsEnabled: merchant.payouts_enabled === 1,
            updatedAt: merchant.updated_at,
          });
        }

        return yield* HttpServerResponse.json(
          { error: "Not found" },
          { status: 404 },
        );
      }),
    };
  }).pipe(
    Effect.provide([
      Cloudflare.D1.QueryDatabaseBinding,
      Stripe.CreateAccountHttp,
      Stripe.CreateAccountLinkHttp,
      Stripe.ConsumeEventsLive,
    ]),
  ),
) {}
```
