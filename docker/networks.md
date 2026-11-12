---
url: https://alchemy.run/docker/networks
title: "Networks"
description: "Connect containers on a Docker.Network so they reach each other by name, cut a network off from the internet, pin its subnet, and attach to networks you didn't create."
access_date: 2026-11-12T09:38:50.977Z
current_date: 2026-11-12T09:38:50.977Z
---

A Docker **network** is a private network on one engine. Containers on the same network reach each other by container name or by an **alias** you give them, on any port, with nothing published to the host. Containers on different networks can’t reach each other at all. Each network gets its own subnet, and Docker runs a DNS server for it, so names resolve to whichever container holds them now.

<svg viewBox="0 0 529.8 312" style="max-width: 609px" role="img" aria-label="On app-network, api reaches Postgres by its alias. The jobs network is internal, so a container that is only on it can't reach the internet, or anything else."><defs><marker id="alc-bd-arrow-1o10xx" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="497.79999999999995" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="156" width="204" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="309.8" y="156" width="203.99999999999994" height="140" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="202" y1="78" x2="323.8" y2="78" marker-end="url(#alc-bd-arrow-1o10xx)"></line><text fill="currentColor" x="264.9" y="70" text-anchor="middle">postgres:5432</text></g> <g><path stroke="currentColor" fill="none" d="M 118 234 V 280 H 305.8"></path><path stroke="currentColor" fill="none" d="M 300.8 273 l 12 14 m 0 -14 l -12 14"></path><text fill="currentColor" x="213.9" y="272" text-anchor="middle">no route out</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">app-network <tspan fill="currentColor" dx="8">bridge</tspan></text> <text fill="currentColor" x="28" y="175" text-anchor="start">jobs <tspan fill="currentColor" dx="8">internal: true</tspan></text> <text fill="currentColor" x="501.79999999999995" y="175" text-anchor="end">outside the engine</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="168" height="48" rx="8"></rect><text fill="currentColor" x="118" y="75" text-anchor="middle">api</text> <text fill="currentColor" x="118" y="91" text-anchor="middle">container</text></g> <g><rect stroke="currentColor" fill="none" x="327.8" y="54" width="168" height="48" rx="8"></rect><text fill="currentColor" x="411.8" y="75" text-anchor="middle">postgres</text> <text fill="currentColor" x="411.8" y="91" text-anchor="middle">alias postgres</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="186" width="168" height="48" rx="8"></rect><text fill="currentColor" x="118" y="207" text-anchor="middle">worker</text> <text fill="currentColor" x="118" y="223" text-anchor="middle">container</text></g> <g><rect stroke="currentColor" fill="none" x="327.8" y="186" width="168" height="48" rx="8"></rect><text fill="currentColor" x="411.8" y="207" text-anchor="middle">registry.npmjs.org</text> <text fill="currentColor" x="411.8" y="223" text-anchor="middle">the internet</text></g></svg> <svg viewBox="0 0 339.2 580" style="max-width: 390px" role="img" aria-label="On app-network, api reaches Postgres by its alias. The jobs network is internal, so a container that is only on it can't reach the internet, or anything else."><defs><marker id="alc-bd-arrow-su6jkm" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor"></path></marker></defs><g><rect stroke="currentColor" fill="none" x="16" y="24" width="204" height="392" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="172" width="204" height="96" rx="12"></rect></g><g><rect stroke="currentColor" fill="none" x="16" y="468" width="204" height="96" rx="12"></rect></g><g><line stroke-opacity="0.2" stroke="currentColor" x1="118" y1="102" x2="118" y2="346" marker-end="url(#alc-bd-arrow-su6jkm)"></line><text fill="currentColor" x="128" y="230">postgres:5432</text></g> <g><path stroke="currentColor" fill="none" d="M 202 226 H 236 V 490 H 224"></path><path stroke="currentColor" fill="none" d="M 213 483 l 14 14 m 0 -14 l -14 14"></path><text fill="currentColor" x="244" y="362">no route out</text></g> <text fill="currentColor" x="28" y="43" text-anchor="start">app-network <tspan fill="currentColor" dx="8">bridge</tspan></text> <text fill="currentColor" x="28" y="191" text-anchor="start">jobs <tspan fill="currentColor" dx="8">internal: true</tspan></text> <text fill="currentColor" x="208" y="487" text-anchor="end">outside the engine</text> <g><rect stroke="currentColor" fill="none" x="34" y="54" width="168" height="48" rx="8"></rect><text fill="currentColor" x="118" y="75" text-anchor="middle">api</text> <text fill="currentColor" x="118" y="91" text-anchor="middle">container</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="350" width="168" height="48" rx="8"></rect><text fill="currentColor" x="118" y="371" text-anchor="middle">postgres</text> <text fill="currentColor" x="118" y="387" text-anchor="middle">alias postgres</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="202" width="168" height="48" rx="8"></rect><text fill="currentColor" x="118" y="223" text-anchor="middle">worker</text> <text fill="currentColor" x="118" y="239" text-anchor="middle">container</text></g> <g><rect stroke="currentColor" fill="none" x="34" y="498" width="168" height="48" rx="8"></rect><text fill="currentColor" x="118" y="519" text-anchor="middle">registry.npmjs.org</text> <text fill="currentColor" x="118" y="535" text-anchor="middle">the internet</text></g></svg>

On app-network, api reaches Postgres by its alias. The jobs network is internal, so a container that is only on it can't reach the internet, or anything else.

## Connect two containers

```typescript
import * as Alchemy from "alchemy";
import * as Docker from "@alchemy.run/docker";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";

export default Alchemy.Stack(
  "MyApp",
  {
    providers: Docker.providers(),
    state: Alchemy.localState(),
  },
  Effect.gen(function* () {
    const network = yield* Docker.Network("app-network");
    const password = yield* Config.Redacted("POSTGRES_PASSWORD");

    yield* Docker.Container("postgres", {
      image: "postgres:18-alpine",
      environment: { POSTGRES_PASSWORD: password },
      networks: [{ name: network.name, aliases: ["postgres"] }],
      start: true,
    });

    const api = yield* Docker.Container("api", {
      image: "ghcr.io/acme/api:latest",
      environment: { DATABASE_HOST: "postgres", DATABASE_PASSWORD: password },
      networks: [{ name: network.name }],
      ports: [{ external: 8080, internal: 3000 }],
      start: true,
    });

    return { api: api.ports };
  }),
);
```

`api` connects to `postgres:5432`. Postgres publishes no port, so only containers on `app-network` can reach it. The alias stays the same when the container is recreated under a new name, which is why you connect by alias.

A container can be on several networks, and `networks` changes apply without recreating it.

## Cut a network off from the internet

```typescript
const jobs = yield* Docker.Network("jobs", { internal: true });
```

An **internal** network has no route out. A container that is only on it can talk to its neighbours and nothing else. A container also on another network can still reach out through that one.

## Choose its addresses

Docker picks each network’s subnet and gateway. Set `ipam` to pin them, for example to avoid a range your VPN uses:

```typescript
const network = yield* Docker.Network("private", {
  ipam: {
    config: [{ subnet: "10.50.0.0/24", gateway: "10.50.0.1", ipRange: "10.50.0.128/25" }],
  },
});
```

Containers then get addresses from `ipRange`. A subnet is a network prefix: `10.50.0.0/24`, not `10.50.0.1/24`. A bad value fails with `InvalidNetworkOptions`, naming the field, before anything is changed.

For IPv6, set `enableIPv6` and declare a subnet of each family:

```typescript
const dualStack = yield* Docker.Network("dual-stack", {
  enableIPv6: true,
  ipam: {
    config: [
      { subnet: "10.51.0.0/24", gateway: "10.51.0.1" },
      { subnet: "fd51:3456:789a::/64", gateway: "fd51:3456:789a::1" },
    ],
  },
});
```

`driver` picks the kind of network: `bridge` by default on one engine, `overlay` across a [swarm](swarm.md), or `macvlan` for addresses on the host’s own network. `driverOpts` passes driver settings through as they are.

## What a change does

A network’s settings are fixed when it’s created, so changing any of them replaces the network. Docker refuses to remove a network while containers are attached, and Alchemy doesn’t disconnect containers it doesn’t manage. Settle a network’s settings before you attach anything long-lived to it.

## Use a network you didn’t create

Attach to a network that Docker Compose or another tool made, by looking it up:

```typescript
const shared = Docker.Network.lookup({ name: "compose_default" });

const api = yield* Docker.Container("api", {
  image: "nginx:alpine",
  networks: [{ name: shared.name }],
});
```

Alchemy reads it and never changes or removes it. To manage an existing network instead, declare it with its `name` and pipe it through `Alchemy.adopt(true)`.
