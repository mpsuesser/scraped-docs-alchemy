---
url: https://alchemy.run/gcp/messaging/eventarc
title: "Eventarc"
description: "React to Firestore changes, Cloud Storage events, and audit-logged Google API calls with GCP.Eventarc.consumeEvents."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Eventarc delivers events from Google services as CloudEvents: a Firestore document was created, an object was written, an API was called. `consumeEvents` creates the trigger and runs your handler for each event.

## Triggers filter events

Google services report what happens in your project as **events**, in the CloudEvents format: a type, a source, attributes, and data. An Eventarc **trigger** delivers the events that match all of its filters to one service.

<svg viewBox="0 0 802.2 215" style="max-width: 923px" role="img" aria-label="A new order document matches every filter of the OrderCreated trigger, so Auditor gets it. The trigger for deletions ignores it."><defs><marker id="alc-fan-arrow-3mxwvf" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="348" y="8" width="230" height="199" rx="10"></rect><text fill="currentColor" x="358" y="26">Eventarc triggers</text></g> <rect stroke="currentColor" fill="none" x="8" y="68" width="288" height="79" rx="8"></rect><text fill="currentColor" x="20" y="88">CloudEvent</text> <text fill="currentColor" x="20" y="108" xml:space="preserve">"type": "…firestore.document.v1.created"</text> <text fill="currentColor" x="20" y="123" xml:space="preserve">"database": "shop"</text> <text fill="currentColor" x="20" y="138" xml:space="preserve">"document": "orders/o_104"</text> <g><path stroke="currentColor" fill="none" d="M 296 107.5 C 327 107.5, 327 77.5, 358 77.5" marker-end="url(#alc-fan-arrow-3mxwvf)"></path><g><rect stroke="currentColor" fill="none" x="360" y="38" width="206" height="79" rx="7"></rect><text fill="currentColor" x="372" y="55">OrderCreated</text> <text fill="currentColor" x="372" y="71" xml:space="preserve">type = …document.v1.created</text> <text fill="currentColor" x="372" y="86" xml:space="preserve">database = shop</text> <text fill="currentColor" x="372" y="101" xml:space="preserve">document matches orders/{id}</text> <line stroke-opacity="0.2" stroke="currentColor" x1="566" y1="77.5" x2="652" y2="77.5" marker-end="url(#alc-fan-arrow-3mxwvf)"></line></g><rect stroke="currentColor" fill="none" x="654" y="54.5" width="140.2" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="724.1" y="74.5" text-anchor="middle" opacity="1">Auditor</text> <text fill="currentColor" x="724.1" y="90.5" text-anchor="middle">Cloud Run service</text></g> <g><path stroke="currentColor" fill="none" d="M 296 107.5 C 327 107.5, 327 163, 358 163"></path><g><rect stroke="currentColor" fill="none" x="360" y="131" width="206" height="64" rx="7"></rect><text fill="currentColor" x="372" y="148">OrderDeleted</text> <text fill="currentColor" x="372" y="164" xml:space="preserve">type = …document.v1.deleted</text> <text fill="currentColor" x="372" y="179" xml:space="preserve">database = shop</text> <text fill="currentColor" x="578" y="167">no match</text></g> <rect stroke="currentColor" fill="none" x="654" y="140" width="140.2" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="724.1" y="160" text-anchor="middle" opacity="0.6">Cleanup</text> <text fill="currentColor" x="724.1" y="176" text-anchor="middle">Cloud Run service</text></g></svg> <svg viewBox="0 0 318 454" style="max-width: 366px" role="img" aria-label="A new order document matches every filter of the OrderCreated trigger, so Auditor gets it. The trigger for deletions ignores it."><defs><marker id="alc-fan-arrow-n-3mxwvf" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="14" y="93" width="304" height="359" rx="10"></rect><text fill="currentColor" x="24" y="111">Eventarc triggers</text></g> <rect stroke="currentColor" fill="none" x="0" y="0" width="288" height="79" rx="8"></rect><text fill="currentColor" x="12" y="20">CloudEvent</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">"type": "…firestore.document.v1.created"</text> <text fill="currentColor" x="12" y="55" xml:space="preserve">"database": "shop"</text> <text fill="currentColor" x="12" y="70" xml:space="preserve">"document": "orders/o_104"</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="79" x2="8" y2="336"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="170.5" x2="20" y2="170.5" marker-end="url(#alc-fan-arrow-n-3mxwvf)"></line><g><rect stroke="currentColor" fill="none" x="22" y="131" width="288" height="79" rx="7"></rect><text fill="currentColor" x="34" y="148">OrderCreated</text> <text fill="currentColor" x="34" y="164" xml:space="preserve">type = …document.v1.created</text> <text fill="currentColor" x="34" y="179" xml:space="preserve">database = shop</text> <text fill="currentColor" x="34" y="194" xml:space="preserve">document matches orders/{id}</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="210" x2="166" y2="238" marker-end="url(#alc-fan-arrow-n-3mxwvf)"></line></g><rect stroke="currentColor" fill="none" x="95.9" y="240" width="140.2" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="260" text-anchor="middle" opacity="1">Auditor</text> <text fill="currentColor" x="166" y="276" text-anchor="middle">Cloud Run service</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="336" x2="20" y2="336"></line><g><rect stroke="currentColor" fill="none" x="22" y="304" width="288" height="64" rx="7"></rect><text fill="currentColor" x="34" y="321">OrderDeleted</text> <text fill="currentColor" x="34" y="337" xml:space="preserve">type = …document.v1.deleted</text> <text fill="currentColor" x="34" y="352" xml:space="preserve">database = shop</text> <text fill="currentColor" x="166" y="387" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="95.9" y="398" width="140.2" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="418" text-anchor="middle" opacity="0.6">Cleanup</text> <text fill="currentColor" x="166" y="434" text-anchor="middle">Cloud Run service</text></g></svg>

A new order document matches every filter of the OrderCreated trigger, so Auditor gets it. The trigger for deletions ignores it.

## React to new Firestore documents

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Shop } from "./resources.ts";

export default class Auditor extends GCP.Function<Auditor>()(
  "Auditor",
  { main: import.meta.url },
  Effect.gen(function* () {
    const shop = yield* Shop;
    const db = yield* GCP.Firestore.WriteDatabase(shop);

    yield* GCP.Eventarc.consumeEvents(
      "OrderCreated",
      {
        location: "us-central1",
        eventFilters: [
          { attribute: "type", value: "google.cloud.firestore.document.v1.created" },
          { attribute: "database", value: shop.databaseId },
          { attribute: "document", value: "orders/{id}", operator: "match-path-pattern" },
        ],
      },
      (event) =>
        db.set(\`audit/${event.id}\`, { document: event.attributes.document ?? "" }).pipe(
          Effect.orDie,
        ),
    );

    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }).pipe(
    Effect.provide(GCP.Firestore.WriteDatabaseHttp),
    Effect.provide(GCP.Run.EventarcEventSource),
  ),
) {}
```

`eventFilters` choose which events you get. `event.attributes` names the document; `event.data` holds the change itself.

## Match the database’s location

Firestore only sends events to triggers in the database’s own location. Pin both:

```typescript
export const Shop = GCP.Firestore.Database("Shop", {
  location: "us-central1",
  type: "FIRESTORE_NATIVE",
});
```

## The service stays private

The trigger signs each delivery with the service’s own identity, and Alchemy grants that identity what it needs to receive events. Nobody else can call the event path.

## Handle repeats

Eventarc delivers at least once. Use `set` keyed by the event, or `create` and ignore `DocumentAlreadyExists`, so a second delivery changes nothing.

## Example

[`examples/gcp-eventarc-firestore`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-eventarc-firestore) writes an audit entry for every new order.

## Where next

- [Firestore](../data/firestore.md) — the database that emits the events.
- [Eventarc reference](https://alchemy.run/providers/gcp/eventarc#trigger) — triggers, channels, and pipelines.
