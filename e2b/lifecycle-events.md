---
url: https://alchemy.run/e2b/lifecycle-events
title: "Lifecycle events"
description: "Handle sandbox lifecycle events in a Worker as E2B sends them."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

E2B reports what happens to each sandbox in the team to a **webhook**: a URL you register, which E2B calls with one signed event each time a sandbox is created, paused, resumed, checkpointed or killed. The event carries the sandbox’s metadata, so your handler can tell whose sandbox it was.

## Each event is a sandbox changing state

<svg viewBox="0 0 647.8 124" style="max-width: 745px" role="img" aria-label="Each change in the sandbox's state is one event, and E2B calls the Worker once for each."><line stroke-opacity="0.2" stroke="currentColor" x1="75.80000000000001" y1="24" x2="75.80000000000001" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="299.8" y1="24" x2="299.8" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="467.8" y1="24" x2="467.8" y2="118"></line><line stroke-opacity="0.2" stroke="currentColor" x1="579.8" y1="24" x2="579.8" y2="118"></line><g><text fill="currentColor" x="4" y="50">user-42</text> <text fill="currentColor" x="4" y="64">sandbox</text> <g><rect stroke="currentColor" fill="none" x="75.80000000000001" y="40" width="224" height="24" rx="5"></rect><text fill="currentColor" x="187.8" y="56" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="299.8" y="40" width="168" height="24" rx="5"></rect><text fill="currentColor" x="383.8" y="56" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="467.8" y="40" width="111.99999999999994" height="24" rx="5"></rect><text fill="currentColor" x="523.8" y="56" text-anchor="middle">running</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="74" x2="647.8" y2="74"></line><text fill="currentColor" x="4" y="94">Events</text> <text fill="currentColor" x="4" y="108">Worker</text> <g><rect stroke="currentColor" fill="none" x="75.80000000000001" y="84" width="33.599999999999994" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="299.8" y="84" width="33.599999999999966" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="467.8" y="84" width="33.60000000000002" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="579.8" y="84" width="33.600000000000136" height="24" rx="5"></rect></g></g><text fill="currentColor" x="75.80000000000001" y="18" text-anchor="middle">created</text> <text fill="currentColor" x="299.8" y="18" text-anchor="middle">paused</text> <text fill="currentColor" x="467.8" y="18" text-anchor="middle">resumed</text> <text fill="currentColor" x="579.8" y="18" text-anchor="middle">killed</text></svg> <svg viewBox="0 0 330 144" style="max-width: 379px" role="img" aria-label="Each change in the sandbox's state is one event, and E2B calls the Worker once for each."><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="24" x2="0" y2="138"></line><line stroke-opacity="0.2" stroke="currentColor" x1="127.2" y1="24" x2="127.2" y2="138"></line><line stroke-opacity="0.2" stroke="currentColor" x1="222.6" y1="24" x2="222.6" y2="138"></line><line stroke-opacity="0.2" stroke="currentColor" x1="286.2" y1="24" x2="286.2" y2="138"></line><g><text fill="currentColor" x="0" y="44">user-42 <tspan fill="currentColor" dx="8">sandbox</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="54" width="127.2" height="24" rx="5"></rect><text fill="currentColor" x="63.6" y="70" text-anchor="middle">running</text></g> <g><rect stroke="currentColor" fill="none" x="127.2" y="54" width="95.39999999999999" height="24" rx="5"></rect><text fill="currentColor" x="174.9" y="70" text-anchor="middle">paused</text></g> <g><rect stroke="currentColor" fill="none" x="222.6" y="54" width="63.599999999999994" height="24" rx="5"></rect><text fill="currentColor" x="254.39999999999998" y="70" text-anchor="middle">running</text></g></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="0" y1="84" x2="330" y2="84"></line><text fill="currentColor" x="0" y="98">Events <tspan fill="currentColor" dx="8">Worker</tspan></text> <g><rect stroke="currentColor" fill="none" x="0" y="108" width="19.08" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="127.2" y="108" width="19.08" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="222.6" y="108" width="19.080000000000013" height="24" rx="5"></rect></g><g><rect stroke="currentColor" fill="none" x="286.2" y="108" width="19.079999999999984" height="24" rx="5"></rect></g></g><text fill="currentColor" x="0" y="18" text-anchor="start">created</text> <text fill="currentColor" x="127.2" y="18" text-anchor="middle">paused</text> <text fill="currentColor" x="222.6" y="18" text-anchor="middle">resumed</text> <text fill="currentColor" x="286.2" y="18" text-anchor="middle">killed</text></svg>

Each change in the sandbox's state is one event, and E2B calls the Worker once for each.

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as E2B from "@alchemy.run/e2b";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Events extends Cloudflare.Worker<Events>()(
  "Events",
  { main: import.meta.url },
  Effect.gen(function* () {
    yield* E2B.consumeSandboxEvents({ events: ["sandbox.lifecycle.killed"] }, (event) =>
      Effect.log(\`${event.sandbox_id} killed: ${event.event_data?.kill_reason}\`),
    );
    return { fetch: Effect.succeed(HttpServerResponse.text("ok")) };
  }).pipe(Effect.provide(E2B.SandboxEventsLive)),
) {}
```

Deploying creates an E2B webhook that points at the Worker. Destroying the Worker deletes it.

## Pick the events you need

`sandbox.lifecycle.created`, `updated`, `paused`, `resumed`, `checkpointed`, and `killed`. Leave out `events` to get all of them:

```typescript
yield* E2B.consumeSandboxEvents({}, (event) =>
  Effect.log(\`${event.type} for ${event.event_data?.sandbox_metadata?.user}\`),
);
```

`event.event_data.sandbox_metadata` holds the sandbox’s metadata, so you can tell whose sandbox it was.

## The Worker checks every delivery

Each request’s `e2b-signature` is checked before your handler runs. An unsigned or wrongly signed request gets a 401. Alchemy creates the signing secret unless you pass `secret`. E2B tries each event up to three times, so keep your handler idempotent on `event.id`.

## Point a webhook at any endpoint

```typescript
export const Lifecycle = E2B.Webhook("Lifecycle", {
  url: "https://hooks.example.com/e2b",
  events: ["sandbox.lifecycle.paused", "sandbox.lifecycle.killed"],
  signatureSecret: Config.Redacted("E2B_WEBHOOK_SECRET"),
});
```

Read its delivery history with `E2B.ReadWebhook(Lifecycle)`: `deliveries()` and `stats()`.
