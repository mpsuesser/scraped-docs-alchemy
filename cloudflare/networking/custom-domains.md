---
url: https://alchemy.run/cloudflare/networking/custom-domains
title: "Custom domains & routes"
description: "Serve Workers from your own domain — create or adopt a Zone, attach custom domains, route hostname patterns, manage DNS records, and control the workers.dev subdomain."
access_date: 2026-11-12T22:28:06.438Z
current_date: 2026-11-12T22:28:06.438Z
---

Every Worker gets a `workers.dev` URL by default, but production apps live on their own domain. That takes a few pieces: the Zone that holds your domain, the `domain` prop that attaches hostnames to a Worker, Routes for pattern-based dispatch, plain DNS records, and the account’s `workers.dev` subdomain.

All the snippets below run inside a Stack’s `Effect.gen` body:

```typescript
import * as Alchemy from "alchemy";
import * as Cloudflare from "@alchemy.run/cloudflare";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  { providers: Cloudflare.providers(), state: Cloudflare.state() },
  Effect.gen(function* () {
    // resources go here
    return {};
  }),
);
```

<svg viewBox="0 0 730.4 242" style="max-width: 840px" role="img" aria-label="Every request to the zone arrives at Cloudflare first. A custom domain or a route decides which Worker answers it, and a hostname with neither goes to its DNS record's origin."><defs><marker id="alc-fan-arrow-pdj218" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="322" y="8" width="204" height="226" rx="10"></rect><text fill="currentColor" x="332" y="26">example.com · zone</text></g> <rect stroke="currentColor" fill="none" x="8" y="96.5" width="262" height="49" rx="8"></rect><text fill="currentColor" x="20" y="116.5">request</text> <text fill="currentColor" x="20" y="136.5" xml:space="preserve">GET https://api.example.com/v1/users</text> <g><path stroke="currentColor" fill="none" d="M 270 121 C 301 121, 301 64, 332 64" marker-end="url(#alc-fan-arrow-pdj218)"></path><g><rect stroke="currentColor" fill="none" x="334" y="38" width="180" height="52" rx="7"></rect><text fill="currentColor" x="346" y="55">custom domain</text> <text fill="currentColor" x="346" y="72.5" xml:space="preserve">api.example.com</text> <line stroke-opacity="0.2" stroke="currentColor" x1="514" y1="64" x2="600" y2="64" marker-end="url(#alc-fan-arrow-pdj218)"></line></g><rect stroke="currentColor" fill="none" x="602" y="41" width="120.39999999999999" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="662.2" y="61" text-anchor="middle" opacity="1">Api</text> <text fill="currentColor" x="662.2" y="77" text-anchor="middle">Worker</text></g> <g><path stroke="currentColor" fill="none" d="M 270 121 C 301 121, 301 130, 332 130"></path><g><rect stroke="currentColor" fill="none" x="334" y="104" width="180" height="52" rx="7"></rect><text fill="currentColor" x="346" y="121">route</text> <text fill="currentColor" x="346" y="138.5" xml:space="preserve">shop.example.com/*</text> <text fill="currentColor" x="526" y="134">no match</text></g> <rect stroke="currentColor" fill="none" x="602" y="107" width="120.39999999999999" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="662.2" y="127" text-anchor="middle" opacity="0.6">Shop</text> <text fill="currentColor" x="662.2" y="143" text-anchor="middle">Worker</text></g> <g><path stroke="currentColor" fill="none" d="M 270 121 C 301 121, 301 196, 332 196"></path><g><rect stroke="currentColor" fill="none" x="334" y="170" width="180" height="52" rx="7"></rect><text fill="currentColor" x="346" y="187">DNS record</text> <text fill="currentColor" x="346" y="204.5" xml:space="preserve">app.example.com → origin</text> <text fill="currentColor" x="526" y="200">no match</text></g> <rect stroke="currentColor" fill="none" x="602" y="173" width="120.39999999999999" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="662.2" y="193" text-anchor="middle" opacity="0.6">your server</text> <text fill="currentColor" x="662.2" y="209" text-anchor="middle">proxied origin</text></g></svg> <svg viewBox="0 0 318 531" style="max-width: 366px" role="img" aria-label="Every request to the zone arrives at Cloudflare first. A custom domain or a route decides which Worker answers it, and a hostname with neither goes to its DNS record's origin."><defs><marker id="alc-fan-arrow-n-pdj218" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="14" y="63" width="304" height="466" rx="10"></rect><text fill="currentColor" x="24" y="81">example.com · zone</text></g> <rect stroke="currentColor" fill="none" x="0" y="0" width="262" height="49" rx="8"></rect><text fill="currentColor" x="12" y="20">request</text> <text fill="currentColor" x="12" y="40" xml:space="preserve">GET https://api.example.com/v1/users</text> <line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="49" x2="8" y2="419"></line><g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="127" x2="20" y2="127" marker-end="url(#alc-fan-arrow-n-pdj218)"></line><g><rect stroke="currentColor" fill="none" x="22" y="101" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="118">custom domain</text> <text fill="currentColor" x="34" y="135.5" xml:space="preserve">api.example.com</text> <line stroke-opacity="0.2" stroke="currentColor" x1="166" y1="153" x2="166" y2="181" marker-end="url(#alc-fan-arrow-n-pdj218)"></line></g><rect stroke="currentColor" fill="none" x="105.80000000000001" y="183" width="120.39999999999999" height="46" rx="8" opacity="1"></rect><text fill="currentColor" x="166" y="203" text-anchor="middle" opacity="1">Api</text> <text fill="currentColor" x="166" y="219" text-anchor="middle">Worker</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="273" x2="20" y2="273"></line><g><rect stroke="currentColor" fill="none" x="22" y="247" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="264">route</text> <text fill="currentColor" x="34" y="281.5" xml:space="preserve">shop.example.com/*</text> <text fill="currentColor" x="166" y="318" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="105.80000000000001" y="329" width="120.39999999999999" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="349" text-anchor="middle" opacity="0.6">Shop</text> <text fill="currentColor" x="166" y="365" text-anchor="middle">Worker</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="8" y1="419" x2="20" y2="419"></line><g><rect stroke="currentColor" fill="none" x="22" y="393" width="288" height="52" rx="7"></rect><text fill="currentColor" x="34" y="410">DNS record</text> <text fill="currentColor" x="34" y="427.5" xml:space="preserve">app.example.com → origin</text> <text fill="currentColor" x="166" y="464" text-anchor="middle">no match</text></g> <rect stroke="currentColor" fill="none" x="105.80000000000001" y="475" width="120.39999999999999" height="46" rx="8" opacity="0.6"></rect><text fill="currentColor" x="166" y="495" text-anchor="middle" opacity="0.6">your server</text> <text fill="currentColor" x="166" y="511" text-anchor="middle">proxied origin</text></g></svg>

Every request to the zone arrives at Cloudflare first. A custom domain or a route decides which Worker answers it, and a hostname with neither goes to its DNS record's origin.

## Create a Zone

A Zone is a domain managed by Cloudflare. Declaring one gives you the zone’s identifiers and the name servers to point your registrar at:

```typescript
const zone = yield* Cloudflare.Zone.Zone("MyZone", {
  name: "example.com",
});
// zone.zoneId, zone.nameServers, zone.accountId, zone.status, ...
```

Zones default to **retain** on removal — destroying the stack does NOT delete the zone in Cloudflare. Opt in to actual deletion by wrapping the resource in `destroy()` from `alchemy/RemovalPolicy`:

```typescript
import { destroy } from "alchemy/RemovalPolicy";

const zone = yield* Cloudflare.Zone.Zone("MyZone", {
  name: "example.com",
}).pipe(destroy());
```

## Adopt an existing Zone

Most domains already exist in Cloudflare before Alchemy enters the picture. A zone carries no ownership markers, so the engine refuses to take over a pre-existing zone unless you opt in with `adopt(true)`:

```typescript
import { adopt } from "alchemy/AdoptPolicy";

const zone = yield* Cloudflare.Zone.Zone("MyZone", {
  name: "example.com",
}).pipe(adopt(true));
```

Without `adopt(true)`, deploying against an existing zone fails with a typed `OwnedBySomeoneElse` error instead of silently clobbering it. Once adopted, the zone behaves like any other managed resource — and it still retains on destroy unless you add `destroy()`.

## Attach a custom domain to a Worker

Pass `domain` — Cloudflare provisions the DNS record and certificate; the zone is inferred from the hostname and must already exist in the account:

```typescript
const worker = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  domain: "api.example.com",
});
// worker.url === "https://api.example.com"
```

`aliases` also serve the Worker; `redirects` answer with a 301 (path and query preserved) to the canonical name, before the Worker runs — ideal for domain migrations:

```typescript
const worker = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  domain: {
    name: "api.example.com",
    aliases: ["api-v2.example.com"],
    redirects: ["api.old-example.com"], // 301 → https://api.example.com
  },
  workersDev: false,
});
// worker.url    === "https://api.example.com"
// worker.urls   === ["https://api.example.com", "https://api-v2.example.com"]
// worker.domain === { name: "api.example.com",
//                     aliases: ["api-v2.example.com"],
//                     redirects: ["api.old-example.com"] }
```

The canonical domain leads `worker.urls` (ahead of `workers.dev`), and redirect hostnames never appear there — they serve no content. Their redirect rules live in the zone’s `http_request_dynamic_redirect` phase, which runs before Workers.

Set `previews: true` so [Worker Previews](../compute/previews.md) of this Worker are served at `<preview-name>.api.example.com`. That is a parent setting — enable it on the production Worker, not on the Preview Worker. Use a dedicated hostname (`previews.example.com`) if production already has subdomains, so the wildcard does not collide.

## Serve a hostname whose DNS is elsewhere

A native custom domain needs the hostname in a Cloudflare zone of the account. When the hostname’s DNS lives in Route 53 or Hetzner DNS, pass that host as `domain.dns`. The Worker then serves the hostname through [Cloudflare for SaaS](https://developers.cloudflare.com/cloudflare-for-platforms/cloudflare-for-saas/start/advanced-settings/worker-as-origin/) on a Cloudflare zone you own:

```typescript
const worker = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  domain: {
    name: "app.example.com",
    zoneId: saasZone.zoneId, // a Cloudflare zone with Cloudflare for SaaS enabled
    cnameTarget: fallback.origin, // the zone's FallbackOrigin hostname
    dns: AWS.Route53.Adapter(),
  },
});
```

Alchemy creates a custom hostname on the SaaS zone, publishes its ownership and certificate `TXT` records and a `CNAME` to `cnameTarget` in Route 53, and routes `app.example.com/*` on the SaaS zone to the Worker. Declare the zone’s `Cloudflare.CustomHostname.FallbackOrigin` once; several Workers can share it. `redirects` and `previews` need a native custom domain. See [DNS Adapters](../../infrastructure-as-code/dns-adapters.md) for the full setup.

## Route a hostname pattern to a Worker

Routes are the classic alternative: a zone-level mapping from a URL pattern to a Worker script. Unlike the `domain` prop, a route can match paths (`example.com/api/*`), so different Workers can serve different parts of one hostname:

```typescript
const worker = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
});

yield* Cloudflare.Workers.WorkerRoute("ApiRoute", {
  zoneId: zone.zoneId,
  pattern: "api.example.com/*",
  script: worker.workerName,
});
```

A route only fires on hostnames that resolve through Cloudflare’s proxy, so pair it with a proxied DNS record. When the Worker is the only origin, an `AAAA 100::` placeholder is the conventional choice:

```typescript
yield* Cloudflare.DNS.Record("ApiPlaceholder", {
  zoneId: zone.zoneId,
  name: "api.example.com",
  type: "AAAA",
  content: "100::",
  proxied: true,
});
```

Cloudflare enforces one route per pattern per zone, and routes carry no ownership markers — so if the route already exists, Alchemy reports it as unowned and refuses to take it over unless you opt in with `adopt(true)` (the same policy as zones):

```typescript
import { adopt } from "alchemy/AdoptPolicy";

yield* Cloudflare.Workers.WorkerRoute("ApiRoute", {
  zoneId: zone.zoneId,
  pattern: "api.example.com/*",
  script: worker.workerName,
}).pipe(adopt(true));
```

Both `pattern` and `script` are mutable — changing either updates the same physical route in place. Changing `zoneId` triggers a replacement.

## Split one hostname across Workers

Routes win over custom domains, so one Worker can own a hostname while others claim paths under it:

```typescript
yield* Cloudflare.Worker("Home", {
  main: "./src/home.ts",
  domain: "api.example.com",
});

yield* Cloudflare.Worker("Orders", {
  main: "./src/orders.ts",
  routes: [{ pattern: "api.example.com/orders*" }],
});
```

`alchemy dev` applies the same routing locally. See [Federated APIs](federated-apis.md) for the full pattern.

## DNS records for non-Worker targets

Not everything on your zone is a Worker. `Cloudflare.DNS.Record` manages individual records — here a plain A record pointing at an external server:

```typescript
yield* Cloudflare.DNS.Record("ApiA", {
  zoneId: zone.zoneId,
  name: "api.example.com",
  type: "A",
  content: "203.0.113.42",
  ttl: 300,
});
```

Proxied records (orange-clouded) route through Cloudflare’s edge — `proxied: true` is only valid for `A`, `AAAA`, and `CNAME` records, and requires the automatic TTL. A common shape is a proxied CNAME at a Cloudflare Tunnel:

```typescript
yield* Cloudflare.DNS.Record("AdminCname", {
  zoneId: zone.zoneId,
  name: "cluster-admin.example.com",
  type: "CNAME",
  content: \`${tunnel.tunnelId}.cfargotunnel.com\`,
  proxied: true,
});
```

DNS records get the same adoption safety as routes: if a record with the same name, type and `content` (and `priority`, for MX and URI) already exists in the zone, Alchemy reports it as unowned and requires `adopt(true)` to take it over. A record with different content is left alone, and a new one is created beside it; only a CNAME, which a name holds just one of, is adopted whatever it points at. This protects hand-edited records — especially apex `A` / `AAAA` and email DKIM/SPF records the dashboard often manages — from being clobbered.

## Disable Workers on a path

A route with no `script` opts matching requests out of Workers entirely. Use it to punch a hole in a broader wildcard route — for example, letting `/assets/*` bypass the Worker and hit the origin directly:

```typescript
yield* Cloudflare.Workers.WorkerRoute("AssetsBypass", {
  zoneId: zone.zoneId,
  pattern: "example.com/assets/*",
});
```

## Control the workers.dev subdomain

Per Worker, the `workersDev` prop toggles the `workers.dev` URLs (it defaults to `true`). Once a custom domain serves production traffic, you may want to switch them off — the custom domain then becomes `worker.url`:

```typescript
const worker = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  domain: "api.example.com",
  workersDev: false,
});
// worker.url === "https://api.example.com"
```

The object form controls the stable URL and per-version preview URLs independently. With the stable URL off and previews on, each deploy’s preview URL (`https://<version-prefix>-<name>.<subdomain>.workers.dev`) becomes `worker.url`:

```typescript
const worker = yield* Cloudflare.Worker("Api", {
  main: "./src/api.ts",
  workersDev: { enabled: false, previewsEnabled: true },
});
```

Account-wide, the `<subdomain>` in `https://<script>.<subdomain>.workers.dev` is a singleton you can pin with the `Subdomain` resource:

```typescript
const sub = yield* Cloudflare.Workers.Subdomain("Subdomain", {
  subdomain: "my-team",
});
// Workers are now served from https://<script>.my-team.workers.dev
```

Subdomain names are globally unique across all Cloudflare accounts; claiming a taken name fails with a typed `SubdomainAlreadyExists` error. Destroy is capture-and-restore: the subdomain is renamed back to whatever the account had before Alchemy first managed it.

## Where next

- [Domains & DNS](domains.md) — the full domain-management surface: zone settings, DNSSEC, certificates, and more.
- [Workers](../compute/workers.md) — the Worker resource itself and its binding system.

Reference:

- [Zone API reference](https://alchemy.run/providers/cloudflare/zone#zone)
- [WorkerRoute API reference](https://alchemy.run/providers/cloudflare/workers#workerroute)
- [Record API reference](https://alchemy.run/providers/cloudflare/dns#record)
- [Subdomain API reference](https://alchemy.run/providers/cloudflare/workers#subdomain)
