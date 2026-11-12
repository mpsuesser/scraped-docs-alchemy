---
url: https://alchemy.run/gcp/networking/private-services
title: "Private services"
description: "Keep a Cloud Run service private and call it from another service with GCP.Run.InvokeService and Google-signed identity tokens."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Cloud Run service is private unless you set `invokerIamDisabled`. Callers need a Google identity token from an account that holds `roles/run.invoker` on it. `InvokeService` sets that up for you.

## How a private call works

Every request to a Cloud Run service passes IAM before it reaches your code. A private service only lets in callers that send a Google identity token for an account holding `roles/run.invoker` on it.

<svg viewBox="0 0 566.4 230" style="max-width: 651px" role="img" aria-label="Cloud Run checks every request before it reaches Quotes. The Gateway's calls carry a token for an account that holds run.invoker, and everything else is refused before your code runs."><defs><marker id="alc-bd-arrow-tcv36y" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="262.4" y="118" width="288" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="148" y1="172" x2="276.4" y2="172" marker-end="url(#alc-bd-arrow-tcv36y)"></line><text fill="currentColor" x="214.2" y="164" text-anchor="middle">identity token</text></g> <g><path stroke="currentColor" fill="none" d="M 148 40 H 466.4 V 114"></path><path stroke="currentColor" fill="none" d="M 459.4 111 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="476.4" y="83">403</text></g> <text fill="currentColor" x="274.4" y="137" text-anchor="start">Cloud Run IAM <tspan fill="currentColor" dx="8">run.invoker: Gateway only</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="82" y="37" text-anchor="middle">anyone</text> <text fill="currentColor" x="82" y="53" text-anchor="middle">no token</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="132" height="48" rx="8"></rect><text fill="currentColor" x="82" y="169" text-anchor="middle">Gateway</text> <text fill="currentColor" x="82" y="185" text-anchor="middle">public service</text></g> <g><rect stroke="currentColor" fill="none" x="280.4" y="148" width="132" height="48" rx="8"></rect><text fill="currentColor" x="346.4" y="169" text-anchor="middle">Quotes</text> <text fill="currentColor" x="346.4" y="185" text-anchor="middle">private service</text></g></svg> <svg viewBox="0 0 383 246" style="max-width: 440px" role="img" aria-label="Cloud Run checks every request before it reaches Quotes. The Gateway's calls carry a token for an account that holds run.invoker, and everything else is refused before your code runs."><defs><marker id="alc-bd-arrow-i1fqkp" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="178" y="120" width="189" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="262" y1="64" x2="262" y2="160" marker-end="url(#alc-bd-arrow-i1fqkp)"></line><text fill="currentColor" x="272" y="118">identity token</text></g> <g><path stroke="currentColor" fill="none" d="M 82 64 V 175 H 174"></path><path stroke="currentColor" fill="none" d="M 171 168 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="92" y="123.5">403</text></g> <text fill="currentColor" x="190" y="139" text-anchor="start">Cloud Run IAM <tspan fill="currentColor" x="190" dy="14">run.invoker: Gateway only</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="82" y="37" text-anchor="middle">anyone</text> <text fill="currentColor" x="82" y="53" text-anchor="middle">no token</text></g> <g><rect stroke="currentColor" fill="none" x="196" y="16" width="132" height="48" rx="8"></rect><text fill="currentColor" x="262" y="37" text-anchor="middle">Gateway</text> <text fill="currentColor" x="262" y="53" text-anchor="middle">public service</text></g> <g><rect stroke="currentColor" fill="none" x="196" y="164" width="132" height="48" rx="8"></rect><text fill="currentColor" x="262" y="185" text-anchor="middle">Quotes</text> <text fill="currentColor" x="262" y="201" text-anchor="middle">private service</text></g></svg>

Cloud Run checks every request before it reaches Quotes. The Gateway's calls carry a token for an account that holds run.invoker, and everything else is refused before your code runs.

## A private backend

Leave `invokerIamDisabled` off:

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default class Quotes extends GCP.Function<Quotes>()(
  "Quotes",
  { main: import.meta.url },
  Effect.gen(function* () {
    return {
      fetch: HttpServerResponse.json({ quote: "Make it work, make it right." }),
    };
  }),
) {}
```

Requests without a valid token get a 403 before they reach your code.

## Call it from another service

```typescript
import * as GCP from "@alchemy.run/gcp";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import Quotes from "./Quotes.ts";

export default class Gateway extends GCP.Function<Gateway>()(
  "Gateway",
  { main: import.meta.url, invokerIamDisabled: true },
  Effect.gen(function* () {
    const quotes = yield* GCP.Run.InvokeService(Quotes);

    return {
      fetch: Effect.gen(function* () {
        const response = yield* quotes.fetch("/quote");
        return HttpServerResponse.text(yield* response.text, {
          status: response.status,
          contentType: "application/json",
        });
      }).pipe(Effect.orDie),
    };
  }).pipe(Effect.provide(GCP.Run.InvokeServiceHttp)),
) {}
```

On deploy, `InvokeService` grants the Gateway `roles/run.invoker` on Quotes and nothing else, and gives the Gateway Quotes’ URL. Each call attaches a fresh identity token. You pass a path, not a URL.

## Send a body

```typescript
const response = yield* orders.fetch("/orders", {
  method: "POST",
  headers: { "content-type": "application/json" },
  body: JSON.stringify({ sku: "abc", quantity: 1 }),
});
const order = yield* response.json;
```

## Handle an unreachable backend

A failed call fails with `GCP.Run.InvokeServiceError`. Turn it into a 502:

```typescript
const response = yield* quotes.fetch("/quote").pipe(
  Effect.catchTag("GCP.Run.InvokeServiceError", (error) =>
    Effect.succeed({
      status: 502,
      text: Effect.succeed(JSON.stringify({ error: error.message })),
    }),
  ),
);
```

## Example

[`examples/gcp-service-to-service`](https://github.com/alchemy-run/alchemy/tree/main/examples/gcp-service-to-service) is a public gateway in front of a private backend.

## Where next

- [VPC & private networking](vpc.md) — reach private IPs from Cloud Run.
- [IAM & bindings](../security/iam.md) — how the invoker grant is scoped.
