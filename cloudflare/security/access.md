---
url: https://alchemy.run/cloudflare/security/access
title: "Protect a Worker with Access"
description: "Put Cloudflare Access in front of a Worker with the access prop — per-Worker policies, shared applications, ctx.access identity at runtime, and a local dev simulation."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Cloudflare Access puts a login wall in front of your Worker — covering its custom domains, routes, `workers.dev` URL, and version preview URLs — and hands your code the authenticated identity at runtime. In alchemy this is a property of the Worker: set the `access` prop.

The prop has two forms — a **dedicated application** owned by the Worker (`access: { policies }`) for per-Worker rules, and a **shared application** (`access: App`) when several Workers sit behind one policy set. Each has its own section below.

<svg viewBox="0 0 561.2 230" style="max-width: 645px" role="img" aria-label="Access stands in front of the Worker on every hostname it serves. Only callers its policies allow get through, and the Worker reads who they are from ctx.access."><defs><marker id="alc-bd-arrow-6v11gv" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="279.8" y="118" width="265.40000000000003" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="205" y1="172" x2="293.8" y2="172" marker-end="url(#alc-bd-arrow-6v11gv)"></line><text fill="currentColor" x="251.4" y="164" text-anchor="middle">identity</text></g> <g><path stroke="currentColor" fill="none" d="M 205 40 H 432.70000000000005 V 114"></path><path stroke="currentColor" fill="none" d="M 425.70000000000005 111 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="442.70000000000005" y="83">login wall</text></g> <text fill="currentColor" x="291.8" y="137" text-anchor="start">Access <tspan fill="currentColor" dx="8">allow emailDomain example.com</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="16" width="189" height="48" rx="8"></rect><text fill="currentColor" x="110.5" y="37" text-anchor="middle">anyone else</text> <text fill="currentColor" x="110.5" y="53" text-anchor="middle">not in the policy</text></g> <g><rect stroke="currentColor" fill="none" x="16" y="148" width="189" height="48" rx="8"></rect><text fill="currentColor" x="110.5" y="169" text-anchor="middle">ana@example.com</text> <text fill="currentColor" x="110.5" y="185" text-anchor="middle">signed in</text></g> <g><rect stroke="currentColor" fill="none" x="297.8" y="148" width="189" height="48" rx="8"></rect><text fill="currentColor" x="392.3" y="169" text-anchor="middle">Admin</text> <text fill="currentColor" x="392.3" y="185" text-anchor="middle">Worker · reads ctx.access</text></g></svg> <svg viewBox="0 0 347 394" style="max-width: 399px" role="img" aria-label="Access stands in front of the Worker on every hostname it serves. Only callers its policies allow get through, and the Worker reads who they are from ctx.access."><defs><marker id="alc-bd-arrow-y44exk" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="268" width="225" height="110" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="128.5" y1="212" x2="128.5" y2="308" marker-end="url(#alc-bd-arrow-y44exk)"></line><text fill="currentColor" x="138.5" y="266">identity</text></g> <g><path stroke="currentColor" fill="none" d="M 223 40 H 257 V 290 H 245"></path><path stroke="currentColor" fill="none" d="M 234 283 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="265" y="169">login wall</text></g> <text fill="currentColor" x="28" y="287" text-anchor="start">Access <tspan fill="currentColor" x="28" dy="14">allow emailDomain example.com</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="37" text-anchor="middle">anyone else</text> <text fill="currentColor" x="128.5" y="53" text-anchor="middle">not in the policy</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="185" text-anchor="middle">ana@example.com</text> <text fill="currentColor" x="128.5" y="201" text-anchor="middle">signed in</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="312" width="189" height="48" rx="8"></rect><text fill="currentColor" x="128.5" y="333" text-anchor="middle">Admin</text> <text fill="currentColor" x="128.5" y="349" text-anchor="middle">Worker · reads ctx.access</text></g></svg>

Access stands in front of the Worker on every hostname it serves. Only callers its policies allow get through, and the Worker reads who they are from ctx.access.

## Dedicated application (per-Worker policies)

The simplest form declares the policies right on the Worker. Alchemy creates a dedicated Access application for it (Cloudflare attaches policies to *applications*, so per-Worker policies mean a per-Worker application — created, updated, and deleted with the Worker, and namespaced under it as `Api/Access`):

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

export default Cloudflare.Worker(
  "Api",
  {
    main: import.meta.url,
    access: {
      policies: [
        { decision: "allow", include: [{ emailDomain: "example.com" }] },
      ],
    },
  },
  Effect.gen(function* () {
    return {
      fetch: Effect.gen(function* () {
        return HttpServerResponse.text("team only");
      }),
    };
  }),
);
```

Unauthenticated requests are redirected to your team’s Access login page before they ever reach the Worker. Anyone with a verified `@example.com` email gets through.

## Read who’s calling

Once a request passes Access, the identity is available from `ctx.access` — one yield away via `Cloudflare.Access.Context`:

```typescript
fetch: Effect.gen(function* () {
  const access = yield* Cloudflare.Access.Context;
  if (access === undefined) {
    return HttpServerResponse.text("Access required", { status: 403 });
  }
  const identity = yield* access.getIdentity();
  return yield* HttpServerResponse.json({
    aud: access.aud,
    email: identity?.email,
    groups: identity?.groups,
  });
}),
```

`access` is `undefined` when the request did not come through Cloudflare Access (for example a route you deliberately left open). `getIdentity()` resolves the user’s email, name, groups, identity provider, device posture, and service-token fields.

## Test locally

In production, `ctx.access` is populated by Cloudflare’s edge after a request passes the Access login wall. Under `alchemy dev` your Worker runs in local workerd — there is no edge and no login — so `ctx.access` would always be `undefined` and the authenticated branch of your handler could never run locally.

`dev.access` stubs that state: when set, every local request behaves as if this one user had logged in — `ctx.access` carries the given `aud` and `getIdentity()` resolves the given identity:

```typescript
export default Cloudflare.Worker(
  "Api",
  {
    main: import.meta.url,
    access: {
      policies: [
        { decision: "allow", include: [{ emailDomain: "example.com" }] },
      ],
    },
    dev: {
      access: {
        aud: "dev",
        identity: { email: "dev@example.com" },
      },
    },
  },
  /* ... */
);
```

Remove the `dev.access` block to simulate unauthenticated requests (`Cloudflare.Access.Context` yields `undefined`), so both branches of your handler are exercisable without deploying. Changing the block restarts the dev server with the new identity. It is inert on deploy: the deployed Worker always gets the real edge-populated `ctx.access`, never the stub.

## Writing policy rules

Rules use Cloudflare’s own shapes, with a scalar shorthand for the single-parameter kinds — both spellings are equivalent:

```typescript
policies: [
  {
    decision: "allow",
    include: [
      { emailDomain: "example.com" },   // { emailDomain: { domain: "example.com" } }
      { email: "sam@partner.com" },     // { email: { email: "sam@partner.com" } }
    ],
    exclude: [{ email: "intern@example.com" }],
    require: [{ geo: "US" }],           // { geo: { countryCode: "US" } }
    sessionDuration: "12h",
  },
]
```

`include` rules combine with OR, `require` with AND, `exclude` with NOT. Parameter-less kinds collapse to bare strings (`"everyone"`, `"certificate"`, `"anyValidServiceToken"`); multi-parameter kinds (`gsuite`, `okta`, `saml`, `oidc`, `azureAD`, `githubOrganization`, …) keep Cloudflare’s wire shape, so anything in the [Access policy docs](https://developers.cloudflare.com/cloudflare-one/policies/access/) pastes in directly.

## Shared application (one policy set, many Workers)

When several Workers should sit behind the same policy set, declare the application once and pass it to each Worker directly:

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

export const TeamOnly = Cloudflare.Access.Application("TeamOnly", {
  type: "self_hosted",
  policies: [
    { decision: "allow", include: [{ emailDomain: "example.com" }] },
  ],
});
```

```typescript
import { TeamOnly } from "./stack.ts";

export default Cloudflare.Worker(
  "Api",
  {
    main: import.meta.url,
    access: TeamOnly,
  },
  /* ... */
);
```

Each enrolled Worker contributes its destinations to the application through the engine’s binding contract, so the application always converges on exactly the set of currently-enrolled Workers — removing the `access` prop (or the Worker itself) un-enrolls it on the next deploy. Keep in mind Access policies are **application-wide**: every Worker enrolled in `TeamOnly` is gated by the same policy set. For per-Worker rules, use the dedicated `{ policies }` form above.

On the dedicated form, set `previews: false` to protect production traffic only, leaving version preview URLs open.

## Protect every Worker on the account

An account-level application covers all current **and future** Workers. Hostname policies beat Worker policies beat account policies, so an individual Worker can still be opened up (or locked down tighter) with its own application:

```typescript
yield* Cloudflare.Access.Application("ProtectAllWorkers", {
  type: "self_hosted",
  destinations: [
    Cloudflare.Access.AllWorkers,
    Cloudflare.Access.AllWorkerPreviews,
  ],
  policies: [
    { decision: "allow", include: [{ emailDomain: "example.com" }] },
  ],
});
```

## Websites too

`access` is part of the shared Worker props, so every `Cloudflare.Website.*` framework accepts it unchanged:

```typescript
const app = yield* Cloudflare.Website.Astro("Docs", {
  access: {
    policies: [
      { decision: "allow", include: [{ emailDomain: "example.com" }] },
    ],
  },
});
```
