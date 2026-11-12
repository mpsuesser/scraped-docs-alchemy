---
url: https://alchemy.run/cloudflare/networking/tunnel
title: "Expose private origins with Tunnel"
description: "Connect a private origin to Cloudflare's edge with a Tunnel, declare its ingress rules as a Configuration resource, and point DNS at it with a proxied CNAME."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Cloudflare Tunnel establishes an outbound connection from your origin — a home lab, a Kubernetes cluster, a VM with no public IP — to Cloudflare’s edge. No open inbound ports, no public address. In alchemy the tunnel, its routing rules, and the DNS record that publishes it are all ordinary resources in one stack.

<svg viewBox="0 0 654.4 294" style="max-width: 753px" role="img" aria-label="cloudflared opens the tunnel from inside your network, so the only way in is through Cloudflare. A visitor's request rides the tunnel down, and an ingress rule sends it to the service."><defs><marker id="alc-bd-arrow-hjvzai" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="228.2" y="24" width="180" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="228.2" y="182" width="410.2" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="160" y1="78" x2="242.2" y2="78" marker-end="url(#alc-bd-arrow-hjvzai)"></line><text fill="currentColor" x="203.1" y="70" text-anchor="middle">GET</text></g> <g><g><rect stroke="currentColor" fill="none" x="296.2" y="179" width="44" height="6"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="296.2" y1="182" x2="340.2" y2="182"></line></g><line stroke-opacity="0.2" stroke="currentColor" x1="318.2" y1="102" x2="318.2" y2="208" marker-end="url(#alc-bd-arrow-hjvzai)"></line><text fill="currentColor" x="328.2" y="161">tunnel</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="390.2" y1="236" x2="472.4" y2="236" marker-end="url(#alc-bd-arrow-hjvzai)"></line><text fill="currentColor" x="433.29999999999995" y="228" text-anchor="middle">ingress</text></g> <g><path stroke="currentColor" fill="none" d="M 88 102 V 230 H 224.2"></path><path stroke="currentColor" fill="none" d="M 221.2 223 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="98" y="170">refused</text></g> <text fill="currentColor" x="240.2" y="43" text-anchor="start">Cloudflare <tspan fill="currentColor" dx="8">edge</tspan></text> <text fill="currentColor" x="626.4" y="201" text-anchor="end">your network <tspan fill="currentColor" dx="8">no inbound ports</tspan></text> <g><rect stroke="currentColor" fill="none" x="16" y="54" width="144" height="48" rx="8"></rect><text fill="currentColor" x="88" y="75" text-anchor="middle">visitor</text> <text fill="currentColor" x="88" y="91" text-anchor="middle">anywhere</text></g> <g><rect stroke="currentColor" fill="none" x="246.2" y="54" width="144" height="48" rx="8"></rect><text fill="currentColor" x="318.2" y="75" text-anchor="middle">app.example.com</text> <text fill="currentColor" x="318.2" y="91" text-anchor="middle">proxied CNAME</text></g> <g><rect stroke="currentColor" fill="none" x="246.2" y="212" width="144" height="48" rx="8"></rect><text fill="currentColor" x="318.2" y="233" text-anchor="middle">cloudflared</text> <text fill="currentColor" x="318.2" y="249" text-anchor="middle">dialed out</text></g> <g><rect stroke="currentColor" fill="none" x="476.4" y="212" width="144" height="48" rx="8"></rect><text fill="currentColor" x="548.4" y="233" text-anchor="middle">api</text> <text fill="currentColor" x="548.4" y="249" text-anchor="middle">localhost:8080</text></g></svg> <svg viewBox="0 0 282.2 552" style="max-width: 325px" role="img" aria-label="cloudflared opens the tunnel from inside your network, so the only way in is through Cloudflare. A visitor's request rides the tunnel down, and an ingress rule sends it to the service."><defs><marker id="alc-bd-arrow-1lz5a0" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="120" width="180" height="110" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="278" width="180" height="258" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="106" y1="64" x2="106" y2="160" marker-end="url(#alc-bd-arrow-1lz5a0)"></line><text fill="currentColor" x="116" y="118">GET</text></g> <g><g><rect stroke="currentColor" fill="none" x="84" y="275" width="44" height="6"></rect><line stroke-opacity="0.2" stroke="currentColor" x1="84" y1="278" x2="128" y2="278"></line></g><line stroke-opacity="0.2" stroke="currentColor" x1="106" y1="212" x2="106" y2="318" marker-end="url(#alc-bd-arrow-1lz5a0)"></line><text fill="currentColor" x="116" y="271">tunnel</text></g> <g><line stroke-opacity="0.2" stroke="currentColor" x1="106" y1="370" x2="106" y2="466" marker-end="url(#alc-bd-arrow-1lz5a0)"></line><text fill="currentColor" x="116" y="424">ingress</text></g> <g><path stroke="currentColor" fill="none" d="M 178 40 H 212 V 300 H 200"></path><path stroke="currentColor" fill="none" d="M 189 293 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="220" y="174">refused</text></g> <text fill="currentColor" x="28" y="139" text-anchor="start">Cloudflare <tspan fill="currentColor" dx="8">edge</tspan></text> <text fill="currentColor" x="184" y="297" text-anchor="end">your network <tspan fill="currentColor" x="184" dy="14">no inbound ports</tspan></text> <g><rect stroke="currentColor" fill="none" x="34" y="16" width="144" height="48" rx="8"></rect><text fill="currentColor" x="106" y="37" text-anchor="middle">visitor</text> <text fill="currentColor" x="106" y="53" text-anchor="middle">anywhere</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="164" width="144" height="48" rx="8"></rect><text fill="currentColor" x="106" y="185" text-anchor="middle">app.example.com</text> <text fill="currentColor" x="106" y="201" text-anchor="middle">proxied CNAME</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="322" width="144" height="48" rx="8"></rect><text fill="currentColor" x="106" y="343" text-anchor="middle">cloudflared</text> <text fill="currentColor" x="106" y="359" text-anchor="middle">dialed out</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="470" width="144" height="48" rx="8"></rect><text fill="currentColor" x="106" y="491" text-anchor="middle">api</text> <text fill="currentColor" x="106" y="507" text-anchor="middle">localhost:8080</text></g></svg>

cloudflared opens the tunnel from inside your network, so the only way in is through Cloudflare. A visitor's request rides the tunnel down, and an ingress rule sends it to the service.

## Create a tunnel

```typescript
import * as Cloudflare from "@alchemy.run/cloudflare";

const tunnel = yield* Cloudflare.Tunnel.Tunnel("MyTunnel");
// Run the connector on your origin:
//   cloudflared tunnel run --token <Redacted.value(tunnel.token)>
```

The resource creates a remotely-managed `cfd_tunnel` and exposes `tunnelId` plus a redacted connector `token` — hand the token to `cloudflared` on the origin and it dials out to the edge. If you omit `name`, a unique one is generated from the app, stage, and logical ID; tunnel names are immutable, so changing the name (or the `tunnelSecret`) triggers a replacement.

## Route hostnames with a Configuration

A remotely-managed tunnel has a single routing document: an ordered list of ingress rules that `cloudflared` matches top-down, first match wins. The `Configuration` resource owns that document:

```typescript
yield* Cloudflare.Tunnel.Configuration("Ingress", {
  tunnelId: tunnel.tunnelId,
  ingress: [
    { hostname: "ui.internal", service: "http://ui.app.svc.cluster.local:80" },
    { hostname: "api.internal", service: "http://api.app.svc.cluster.local:8080" },
  ],
});
```

`service` is the upstream each hostname forwards to — a private IP, a Kubernetes Service DNS name, or a pseudo-service like `http_status:404`. The required catch-all rule (a final entry with no hostname) is appended automatically, since Cloudflare rejects a config whose last rule has a hostname; override what it returns with `catchAllService: "http_status:503"`.

## Point DNS at the tunnel

Publishing a hostname through the tunnel is a proxied CNAME to `<tunnelId>.cfargotunnel.com`:

```typescript
yield* Cloudflare.DNS.Record("AppCname", {
  zoneId: zone.zoneId,
  name: "app.example.com",
  type: "CNAME",
  content: \`${tunnel.tunnelId}.cfargotunnel.com\`,
  proxied: true,
});
```

`proxied: true` is what makes it work — traffic must enter Cloudflare’s edge to be forwarded down the tunnel. See [Domains & DNS](domains.md) for creating or adopting the `zone` this record lives in.

## Private hostnames and networks

For origins that should only be reachable by WARP clients — no public DNS at all — route the hostname through the tunnel with a `HostnameRoute`:

```typescript
yield* Cloudflare.Tunnel.HostnameRoute("AppRoute", {
  hostname: "app.internal.example.com",
  tunnelId: tunnel.tunnelId,
  comment: "internal wiki behind the datacenter tunnel",
});
```

Hostname, tunnel, and comment are all mutable in place — updates converge on the same route rather than replacing it. Hostnames are unique per account.

To route entire private CIDR ranges — including overlapping ones — create a `VirtualNetwork` and target it from a `Route`:

```typescript
const vnet = yield* Cloudflare.Tunnel.VirtualNetwork("Staging", {
  comment: "staging private network",
});

yield* Cloudflare.Tunnel.Route("StagingNet", {
  tunnelId: tunnel.tunnelId,
  network: "10.4.0.0/16",
  virtualNetworkId: vnet.virtualNetworkId,
});
```

Each virtual network is an isolated routing namespace, so two environments can both claim `10.4.0.0/16` and WARP clients pick which copy they see. Passing `vnet.virtualNetworkId` into the `Route` also gets destroy ordering right — a virtual network can’t be deleted while routes still reference it.

## Where next

- [Domains & DNS](domains.md) — the zone and records the tunnel CNAME lives in.
- [Tunnel API reference](https://alchemy.run/providers/cloudflare/tunnel#tunnel) — including the `ReadTunnel` / `WriteTunnel` / `ReadWriteTunnel` bindings for managing tunnels from inside a deployed Worker.
- [Configuration API reference](https://alchemy.run/providers/cloudflare/tunnel#configuration)
- [HostnameRoute API reference](https://alchemy.run/providers/cloudflare/tunnel#hostnameroute)
- [VirtualNetwork API reference](https://alchemy.run/providers/cloudflare/tunnel#virtualnetwork)
