---
url: https://alchemy.run/cloudflare/email
title: "Email"
description: "Cloudflare Email Routing turns a zone into a mail endpoint — enable routing, verify destinations, forward with rules and a catch-all, hand mail to a Worker, and send from Workers with the send_email binding."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Email Routing turns a zone you own into a mail endpoint: rules match inbound mail and forward it, drop it, or hand it to a Worker — and once routing is enabled, Workers can also *send* mail from the domain through a `send_email` binding.

You need a zone first — see [Domains & DNS](networking/domains.md). Receiving needs a verified destination address (or a Worker); sending needs routing enabled on the sender’s domain. The category also covers sending subdomains and the enterprise Email Security suite, both below.

<svg viewBox="0 0 726.5 242" style="max-width: 835px" role="img" aria-label="Rules match each inbound message in priority order and forward it, hand it to a Worker, or drop it. Mail no rule matches falls to the catch-all."><defs><marker id="alc-fan-arrow-xwat8l" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="257" y="8" width="197.5" height="226" rx="10"></rect><text fill="currentColor" x="267" y="26">example.com · Email Routing</text></g> <rect stroke="currentColor" fill="none" x="8" y="89" width="197" height="64" rx="8"></rect><text fill="currentColor" x="20" y="109">inbound mail</text> <text fill="currentColor" x="20" y="129" xml:space="preserve">to: inbox@example.com</text> <text fill="currentColor" x="20" y="144" xml:space="preserve">from: ana@customer.example</text> <g><path stroke="currentColor" fill="none" d="M 205 121 C 236 121, 236 64, 267 64" marker-end="url(#alc-fan-arrow-xwat8l)"></path><g><rect stroke="currentColor" fill="none" x="269" y="38" width="173.5" height="52" rx="7"></rect><text fill="currentColor" x="281" y="55">rule</text> <text fill="currentColor" x="281" y="72.5" xml:space="preserve">to = inbox@example.com</text> <line stroke-opacity="0.2" stroke="currentColor" x1="442.5" y1="64" x2="528.5" y2="64" marker-end="url(#alc-fan-arrow-xwat8l)"></line></g><rect stroke="currentColor" fill="none" x="530.5" y="41" width="188" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="624.5" y="61" text-anchor="middle" opacity="1">you@personal.example</text> <text fill="currentColor" x="624.5" y="77" text-anchor="middle">forward</text></g> <g><path stroke="currentColor" fill="none" d="M 205 121 C 236 121, 236 130, 267 130"></path><g><rect stroke="currentColor" fill="none" x="269" y="104" width="173.5" height="52" rx="7"></rect><text fill="currentColor" x="281" y="121">rule</text> <text fill="currentColor" x="281" y="138.5" xml:space="preserve">to = orders@example.com</text> <text fill="currentColor" x="454.5" y="134">no match</text></g> <rect stroke="currentColor" fill="none" x="530.5" y="107" width="188" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="624.5" y="127" text-anchor="middle" opacity="0.6">Orders</text> <text fill="currentColor" x="624.5" y="143" text-anchor="middle">Worker · email handler</text></g> <g><path stroke="currentColor" fill="none" d="M 205 121 C 236 121, 236 196, 267 196"></path><g><rect stroke="currentColor" fill="none" x="269" y="170" width="173.5" height="52" rx="7"></rect><text fill="currentColor" x="281" y="187">catch-all</text> <text fill="currentColor" x="281" y="204.5" xml:space="preserve">anything unmatched</text> <text fill="currentColor" x="454.5" y="200">no match</text></g> <rect stroke="currentColor" fill="none" x="530.5" y="173" width="188" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="624.5" y="193" text-anchor="middle" opacity="0.6">drop</text> <text fill="currentColor" x="624.5" y="209" text-anchor="middle">discard</text></g></svg> <svg viewBox="0 0 318 546" style="max-width: 366px" role="img" aria-label="Rules match each inbound message in priority order and forward it, hand it to a Worker, or drop it. Mail no rule matches falls to the catch-all."><defs><marker id="alc-fan-arrow-n-xwat8l" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="14" y="78" width="304" height="466" rx="10"></rect><text fill="currentColor" x="24" y="96">example.com · Email Routing</text></g> <rect stroke="currentColor" fill="none" x="0" y="0" width="197" height="64" rx="8"></rect><text fill="currentColor" x="12" y="20">inbound mail</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">to: inbox@example.com</text> <text fill="currentColor" x="12" y="55" xml:space="preserve">from: ana@customer.example</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="64" x2="8" y2="434"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="142" x2="20" y2="142" marker-end="url(#alc-fan-arrow-n-xwat8l)"></line><g><rect stroke="currentColor" fill="none" x="22" y="116" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="133">rule</text> <text fill="currentColor" x="34" y="150.5" xml:space="preserve">to = inbox@example.com</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="168" x2="166" y2="196" marker-end="url(#alc-fan-arrow-n-xwat8l)"></line></g><rect stroke="currentColor" fill="none" x="72" y="198" width="188" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="218" text-anchor="middle" opacity="1">you@personal.example</text> <text fill="currentColor" x="166" y="234" text-anchor="middle">forward</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="288" x2="20" y2="288"></line><g><rect stroke="currentColor" fill="none" x="22" y="262" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="279">rule</text> <text fill="currentColor" x="34" y="296.5" xml:space="preserve">to = orders@example.com</text> <text fill="currentColor" x="166" y="333" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="72" y="344" width="188" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="364" text-anchor="middle" opacity="0.6">Orders</text> <text fill="currentColor" x="166" y="380" text-anchor="middle">Worker · email handler</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="434" x2="20" y2="434"></line><g><rect stroke="currentColor" fill="none" x="22" y="408" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="425">catch-all</text> <text fill="currentColor" x="34" y="442.5" xml:space="preserve">anything unmatched</text> <text fill="currentColor" x="166" y="479" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="72" y="490" width="188" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="510" text-anchor="middle" opacity="0.6">drop</text> <text fill="currentColor" x="166" y="526" text-anchor="middle">discard</text></g></svg>

Rules match each inbound message in priority order and forward it, hand it to a Worker, or drop it. Mail no rule matches falls to the catch-all.

## Enable routing on the zone

`Email.Routing` is the prerequisite for both directions:

```typescript
const routing = yield* Cloudflare.Email.Routing("Routing", {
  zone: "example.com",
});
// routing.zoneId, routing.enabled, routing.status
```

`zone` accepts a zone id, a zone name, or a `{ zoneId }` object; moving zones is a replacement, and destroy disables Email Routing on the zone.

## Verify a destination address

Rules can only forward to addresses Cloudflare has verified:

```typescript
const inbox = yield* Cloudflare.Email.Address("Inbox", {
  email: "you@personal.example",
});
// inbox.verified, inbox.verifiedAt
```

Cloudflare emails a verification link on first create — until it’s clicked, forwards don’t deliver; addresses are account-scoped, and changing `email` is a replacement.

## Forward an address

`Email.Rule` matches inbound mail on the zone and applies actions:

```typescript
const rule = yield* Cloudflare.Email.Rule("InboxRule", {
  zone: "example.com",
  name: "Forward inbox to destination",
  matchers: [{ type: "literal", field: "to", value: "inbox@example.com" }],
  actions: [{ type: "forward", value: ["you@personal.example"] }],
});
```

Mail that matches no rule falls through to the zone’s catch-all; lower `priority` numbers run first (default `0`).

## Catch everything else

The catch-all is a per-zone singleton — `Email.CatchAll` puts the desired configuration and restores the prior state on destroy:

```typescript
yield* Cloudflare.Email.CatchAll("CatchAll", {
  zone: routing.zoneId,
  actions: [{ type: "forward", value: ["you@personal.example"] }],
});
```

To silently discard unmatched mail instead:

```typescript
yield* Cloudflare.Email.CatchAll("DropTheRest", {
  zone: routing.zoneId,
  actions: [{ type: "drop" }],
});
```

## Hand mail to a Worker

Both `Rule` and `CatchAll` accept a `worker` action naming a deployed Worker script (its `workerName` attribute):

```typescript
yield* Cloudflare.Email.CatchAll("CatchAllWorker", {
  zone: routing.zoneId,
  actions: [{ type: "worker", value: ["my-email-worker"] }],
});
```

Matched messages are delivered to that Worker’s `email` handler instead of a mailbox.

## Declare the send\_email binding

`Email.SendEmail` is a binding descriptor, not a cloud resource — it names the binding and pins sender/destination restrictions Cloudflare enforces at send time:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const Email = Cloudflare.Email.SendEmail("Email", {
  allowedSenderAddresses: ["bot@example.com"],
  destinationAddress: "you@personal.example",
});
```

`destinationAddress` pins one verified destination, `allowedDestinationAddresses` allows a list (the two are mutually exclusive); omit all three to send to any verified destination on the account.

## Send from a Worker

Bind the descriptor with `Email.Send` in the Worker’s Construction phase and provide `Email.SendBinding` as the implementation layer:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Email } from "./email.ts";

export default Cloudflare.Worker(
  "Notifier",
  { main: import.meta.url },
  Effect.gen(function* () {
    const email = yield* Cloudflare.Email.Send(Email);

    return {
      fetch: Effect.gen(function* () {
        yield* email.send({
          from: "bot@example.com",
          to: "you@personal.example",
          subject: "Hello from a Worker",
          text: "Sent via the send_email binding.",
        });
        return yield* HttpServerResponse.json({ ok: true });
      }).pipe(
        Effect.catchTag("SendEmailError", (error) =>
          Effect.succeed(
            HttpServerResponse.text(error.message, { status: 502 }),
          ),
        ),
      ),
    };
  }).pipe(Effect.provide(Cloudflare.Email.SendBinding)),
);
```

Rejections (an unverified destination, a sender outside the allow list) surface as a typed `SendEmailError`; the client also exposes `sendRaw` for a pre-built `EmailMessage` and `raw` for the underlying runtime binding. The `from:` address must be on a domain with routing enabled, and the `to:` address must be a verified destination.

## Send from a subdomain

`Email.SendingSubdomain` registers e.g. `mail.example.com` as a sender domain without touching the apex:

```typescript
const sending = yield* Cloudflare.Email.SendingSubdomain("Mail", {
  zoneId: zone.zoneId,
  name: "mail.example.com",
});
// sending.enabled — true once DNS records validated
```

For zones not on Cloudflare DNS, fetch the expected records via `emailSending.getSubdomainDns` and create them at your host — `enabled` flips to `true` once they validate.

## Email Security (enterprise)

The Email Security add-on filters inbound mail before delivery; accounts without the entitlement get a typed `EmailSecurityNotEntitled` error:

```typescript
yield* Cloudflare.Email.BlockSender("KnownPhisher", {
  pattern: "phisher@malicious.example.com",
  patternType: "EMAIL",
});
```

The rest of the suite:

- [Domain](https://alchemy.run/providers/cloudflare/email#domain) — per-domain dispositions and TLS enforcement.
- [AllowPolicy](https://alchemy.run/providers/cloudflare/email#allowpolicy) — acceptable, trusted, and exempt senders.
- [TrustedDomain](https://alchemy.run/providers/cloudflare/email#trusteddomain) — suppress lookalike and new-domain detections.
- [ImpersonationRegistryEntry](https://alchemy.run/providers/cloudflare/email#impersonationregistryentry) — protect display names from BEC.

## Where next

The full walkthrough:

- [Send & receive email](email/send-and-receive.md) — wire both directions on one zone, end to end.

Related:

- [Workers](compute/workers.md) — the runtime that sends and receives.
- [Domains & DNS](networking/domains.md) — the zone mail routes through.
- [Custom domains & routes](networking/custom-domains.md) — get or adopt the zone first.

Reference:

- [Routing API reference](https://alchemy.run/providers/cloudflare/email#routing)
- [Address API reference](https://alchemy.run/providers/cloudflare/email#address)
- [Rule API reference](https://alchemy.run/providers/cloudflare/email#rule)
- [CatchAll API reference](https://alchemy.run/providers/cloudflare/email#catchall)
- [Send API reference](https://alchemy.run/providers/cloudflare/email#send)
- [SendingSubdomain API reference](https://alchemy.run/providers/cloudflare/email#sendingsubdomain)
- [Domain API reference](https://alchemy.run/providers/cloudflare/email#domain)
- [BlockSender API reference](https://alchemy.run/providers/cloudflare/email#blocksender)
- [AllowPolicy API reference](https://alchemy.run/providers/cloudflare/email#allowpolicy)
- [TrustedDomain API reference](https://alchemy.run/providers/cloudflare/email#trusteddomain)
- [ImpersonationRegistryEntry API reference](https://alchemy.run/providers/cloudflare/email#impersonationregistryentry)
