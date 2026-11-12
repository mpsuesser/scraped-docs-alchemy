---
url: https://alchemy.run/modal/services/web-endpoints
title: "Web endpoints"
description: "Serve a Modal Function on the web at a modal.run URL."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

A Function with `web` serves its `fetch` handler at a `*.modal.run` URL. Modal starts containers as requests arrive, like it does for calls.

## Proxy auth keeps strangers out

A web endpoint answers anyone unless it requires proxy auth. Then Modal’s proxy checks each request’s token before it reaches a container:

<svg viewBox="0 0 652.5999999999999 230" style="max-width: 750px" role="img" aria-label="With requiresProxyAuth, a request reaches Site only with a webhook token's Modal-Key and Modal-Secret headers. FetchServer adds them for the Worker."><defs><marker id="alc-bd-arrow-yblas2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="352.4" y="118" width="284.19999999999993" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="172" y1="172" x2="366.4" y2="172" marker-end="url(#alc-bd-arrow-yblas2)"></line><text fill="currentColor" x="271.2" y="164" text-anchor="middle">Modal-Key · Modal-Secret</text></g> <g><path stroke="currentColor" fill="none" d="M 172 40 H 540.5999999999999 V 114"></path><path stroke="currentColor" fill="none" d="M 533.5999999999999 111 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="550.5999999999999" y="83">401</text></g> <text fill="currentColor" x="364.4" y="137" text-anchor="start">acme--site.modal.run <tspan fill="currentColor" dx="8">requiresProxyAuth</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="156" height="48" rx="8"></rect><text fill="currentColor" x="94" y="37" text-anchor="middle">anyone</text> <text fill="currentColor" x="94" y="53" text-anchor="middle">no token</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="156" height="48" rx="8"></rect><text fill="currentColor" x="94" y="169" text-anchor="middle">Api</text> <text fill="currentColor" x="94" y="185" text-anchor="middle">Worker · FetchServer</text></g> <g><rect stroke="currentColor" fill="none" x="370.4" y="148" width="156" height="48" rx="8"></rect><text fill="currentColor" x="448.4" y="169" text-anchor="middle">Site</text> <text fill="currentColor" x="448.4" y="185" text-anchor="middle">Function · web</text></g></svg> <svg viewBox="0 0 296.4 394" style="max-width: 341px" role="img" aria-label="With requiresProxyAuth, a request reaches Site only with a webhook token's Modal-Key and Modal-Secret headers. FetchServer adds them for the Worker."><defs><marker id="alc-bd-arrow-5i3eua" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="268" width="192" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="112" y1="212" x2="112" y2="308" marker-end="url(#alc-bd-arrow-5i3eua)"></line><text fill="currentColor" x="122" y="266">Modal-Key · Modal-Secret</text></g> <g><path stroke="currentColor" fill="none" d="M 190 40 H 224 V 290 H 212"></path><path stroke="currentColor" fill="none" d="M 201 283 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="232" y="169">401</text></g> <text fill="currentColor" x="28" y="287" text-anchor="start">acme--site.modal.run <tspan fill="currentColor" x="28" dy="14">requiresProxyAuth</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="37" text-anchor="middle">anyone</text> <text fill="currentColor" x="112" y="53" text-anchor="middle">no token</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="185" text-anchor="middle">Api</text> <text fill="currentColor" x="112" y="201" text-anchor="middle">Worker · FetchServer</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="156" height="48" rx="8"></rect><text fill="currentColor" x="112" y="333" text-anchor="middle">Site</text> <text fill="currentColor" x="112" y="349" text-anchor="middle">Function · web</text></g></svg>

With requiresProxyAuth, a request reaches Site only with a webhook token's Modal-Key and Modal-Secret headers. FetchServer adds them for the Worker.

## Serve fetch

```typescript
import * as Modal from "@alchemy.run/modal";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";
import { Inference } from "./Inference.ts";

export default class Site extends Modal.Function<Site>()(
  "Site",
  { main: import.meta.url, app: Inference, web: true },
  Effect.gen(function* () {
    return { fetch: Effect.succeed(HttpServerResponse.text("hello from Modal")) };
  }),
) {}
```

The URL is the `url` attribute.

## Pick the label in its URL

```typescript
{ main: import.meta.url, app: Inference, web: { label: "site" } }
```

The URL becomes `https://<workspace>--site.modal.run`.

## Require proxy auth

```typescript
{ main: import.meta.url, app: Inference, web: { requiresProxyAuth: true } }
```

Callers then send the `Modal-Key` and `Modal-Secret` headers of a [`Modal.WebhookToken`](../security/access-control.md).

## Call it from a Worker

```typescript
const site = yield* Modal.FetchServer(Site);
const response = yield* site.fetch(HttpClientRequest.get("/"));
```

`FetchServer` adds the proxy auth headers for you.
