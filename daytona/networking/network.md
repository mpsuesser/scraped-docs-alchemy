---
url: https://alchemy.run/daytona/networking/network
title: "Network and previews"
description: "What a Daytona sandbox can reach — your organization's tier, domain and CIDR allowlists, no network at all — and how preview URLs let the outside reach a port inside it."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

Daytona controls a sandbox’s network from outside the sandbox, so nothing running inside it can turn the limits off. Two layers decide what goes out. Your organization’s **tier** sets the floor: below Tier 3, sandboxes reach only an allowlist of essential services, such as package registries, GitHub, AI APIs and Daytona’s own API, and from Tier 3 they reach the whole internet. On top of that, each sandbox can narrow it further.

## Your tier sets the floor, each sandbox narrows it

<svg viewBox="0 0 413.4 268" style="max-width: 475px" role="img" aria-label="DevBox allows github.com and registry.npmjs.org. A clone from GitHub goes through; a connection to any host not on the list is refused before it leaves Daytona."><defs><marker id="alc-bd-arrow-pevlbx" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="232.79999999999998" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="156" width="168" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="78" x2="261.4" y2="78" marker-end="url(#alc-bd-arrow-pevlbx)"></line><text fill="currentColor" x="215.7" y="70" text-anchor="middle">git clone</text></g> <g><path stroke="currentColor" fill="none" d="M 166 78 H 100 V 152"></path><path stroke="currentColor" fill="none" d="M 93 149 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="110" y="121">refused</text></g> <text fill="currentColor" x="236.79999999999998" y="43" text-anchor="end">sandbox <tspan fill="currentColor" dx="8">limits enforced outside</tspan></text> <text fill="currentColor" x="172" y="175" text-anchor="end">elsewhere</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="75" text-anchor="middle">DevBox</text> <text fill="currentColor" x="100" y="91" text-anchor="middle">domainAllowList</text></g> <g><rect stroke="currentColor" fill="none" x="265.4" y="54" width="132" height="48" rx="8"></rect><text fill="currentColor" x="331.4" y="75" text-anchor="middle">github.com</text> <text fill="currentColor" x="331.4" y="91" text-anchor="middle">allowed</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="186" width="132" height="48" rx="8"></rect><text fill="currentColor" x="100" y="207" text-anchor="middle">example.com</text> <text fill="currentColor" x="100" y="223" text-anchor="middle">not on the list</text></g></svg>

DevBox allows github.com and registry.npmjs.org. A clone from GitHub goes through; a connection to any host not on the list is refused before it leaves Daytona.

## Allow only some domains

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";

export const DevBox = Daytona.Sandbox("DevBox", {
  snapshot: Base,
  domainAllowList: ["github.com", "registry.npmjs.org"],
});
```

Use `networkAllowList` for CIDR ranges, and `networkBlockAll: true` to cut the network entirely. All three change in place.

## Route traffic through your own proxy

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";

export const DevBox = Daytona.Sandbox("DevBox", {
  snapshot: Base,
  domainAllowList: ["github.com", "registry.npmjs.org"],
  outboundProxyUrl: "http://proxy.example.com:3128",
});
```

Daytona sets `HTTP_PROXY` and `HTTPS_PROXY` in the sandbox to this URL, and enforces it at the network only when the sandbox also has a `domainAllowList`. It’s a forward proxy: HTTPS requests reach it as `CONNECT` tunnels, so it runs on a host that accepts them, such as a Fly Machine. A Cloudflare Worker can’t serve `CONNECT`, so it can’t be one. The URL is in the sandbox’s environment, so the sandbox can read any password in it. To add a credential the sandbox never sees, use a [Secret](../security/secrets.md#declare-a-secret-for-some-hosts). Changing the URL replaces the sandbox.

## Limit sandboxes you create

```typescript
const sandbox = yield* sandboxes.create({
  domainAllowList: "api.openai.com,github.com",
  networkBlockAll: false,
});
yield* sandbox.updateNetworkSettings({ domainAllowList: "github.com" });
```

`create` takes Daytona’s own request, so the lists are comma-separated strings here.

## Reach a port from outside

A **preview URL** reaches a port inside a sandbox through Daytona’s proxy. A private sandbox’s previews need its preview token in the `X-Daytona-Preview-Token` header; a `public` sandbox’s previews are open to anyone.

A request reaches port 3000 only through the preview URL, and Daytona's proxy lets it through only with the sandbox's preview token. A public sandbox skips the check.

```typescript
const preview = yield* sandbox.getPortPreviewUrl({ port: 3000 });
const signed = yield* sandbox.getSignedPortPreviewUrl({ port: 3000, expiresInSeconds: 3600 });
const port = yield* sandbox.getTcpPort(3000);
const res = yield* port.fetch(HttpClientRequest.get("/health"));
```

A signed URL works without the token until it expires, so you can hand it to a browser. `getTcpPort` returns the port’s preview `url` and `token`, and its `fetch` sends a request there with the token for you.

## Put a Worker in front of a port

`fetch` also takes the request your Worker received, and returns a response the Worker can send straight back:

```typescript
const port = yield* sandbox.getTcpPort(3000);
return yield* port.fetch(yield* HttpServerRequest);
```

The method, path, query, headers and body go to the sandbox’s server, and its status, headers and body come back. Bodies stream both ways.

## Open previews to anyone

```typescript
import * as Daytona from "@alchemy.run/daytona";
import { Base } from "./Images.ts";

export const DevBox = Daytona.Sandbox("DevBox", { snapshot: Base, public: true });
```

`public` changes in place.

Credentials a sandbox can use but not read are in [Secrets](../security/secrets.md).
